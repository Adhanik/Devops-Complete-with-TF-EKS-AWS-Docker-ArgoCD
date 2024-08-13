
We have a go-web-app application developed by developers, and they have reached out to us to implement devops. 

# What we will be doing

1. We will be containerising the project. For this we will write a multi stage Docker file, we will implement images with reduced size, as well as security.

2. We will create KB manifest, such as Deployemnt, Service and Ingress.
3. Then we will setup CI using Github Actions.
4. We will implement CD using GitOps, using ArgoCD to implement continouous Delivery.
5. We will setup a KB cluster, CI/CD has to deploy the application on target platform, and target platform is KB. 
6. We will create a EKS cluster
7. Set up Helm chart for application. This is done so that in future if development team wants to deploy this application on to dev, QZ, production, instead of writing manifests file for each evn, they can use the Helm chart and pass the values.yml
8. We will setup Ingress controller configuration so that IC would create a Load Balancer depending upon the Ingress configuration, so that application is exposed to outside world.
9. We will also learn how to map the LB ip address to local DNS, so we can tst if application is accessbile from outside world.

# PREREQUISITE

Install Docker locally
Before we start with containerisation, we need to understand the project. We should know how the application should be built, how application needs to be run, on which port the application is running, and how does the application looks like.

So firstly we will clone the repo, and test the application in our local.

Firstly we will install Go on our mac - brew install go
Now before running the application, we need to build the application. The command to build is 

  -- go build -o main   - where main is the build binary
  -- ./main            - To execute a go binary

The application is accessible on path - localhost:8080/courses

Now since we have understood how to run the application locally, build and run the binary, we will start writing the docker file for this.

## Implementing Containerisation

# Docker file

We will write a multi stage Docker file. There are various advantages to writing a multi stage Docker file, 

    In stage 1, you can build docker image, use dependencies, and once application is build, in stage 2, you can use distroless image as your base image. This distroless image will add capabilities like security and reduced image size.

    In stage 2, we will copy the binary that is build in stage 1, expose the port, and run the application.

Always mention that stage2, which is the final stage, to make use of Distroless image.

# Building docker file

We will choose golang as the base image. We can make use of ubuntu, alpine as well, but the only difference is that we again need to install go on that base images, because the application is written in go programming language. We need to have go installed on your base image. so instead of doing this, we are directly using golang image.

## Creating alias

Since we are going to write a multi stage docker file, we will create an alias for first stage. 
Definition - The name of this build stage. Set the base image to use for any subsequent instructions that follow and also give this build stage a name.

  FROM golang:1.21 as base

We will create a workdir, and all the commands written in our Dockerfile will be executed in this workdir only.
Definition - The absolute or relative path to use as the working directory. Will be created if it does not exist.
Set the working directory for any ADD, COPY, CMD, ENTRYPOINT, or RUN instructions that follow.

  WORKDIR /app

We will copy go.mod file in our workdir, it is required bcoz the dependencies for a go app are stored in go.mod file. We will run the same, consider go.md like requirements.txt and thn you run pip install -r requirements.txt to install those dependencies, the same we are doing here.

  COPY go.mod .
  RUN go mod download

RUN command parameter ...
The command to run.
Execute commands inside a shell.


Now we will copy the entire source code in docker image

COPY [flags] source ... dest
The resource to copy.

Copy new files and directories to the image's filesystem.

  COPY . .

We will run the comand to build binary (we are in /app dir)

  RUN go build -o main .

After the above command is executed, the artifact/binary called main will be created in Dockerimage. We can now expost it on our local port and give CMD to execute the binary

  EXPOSE 8080
  CMD ["./main"]

But our main motive is that the docker image made is of reduced size, and secure in nature. For that, we need to use is a distorless image

# Final stage - Distroless image

  FROM gcr.io/distroless/base   (this is a distroless image, similar to goland image we used - FROM golang:1.21 as base )

We will copyt the binary on to distroless image.

 COPY --from=base /app/main .  (base is alias for our first stage, from  base stage, copy binary(main) which is in /app dir to default dir(represented by .))

We need to copy also the static files in our distroless image
  
  COPY --from=base /app/static  ./static (copy static files to a static dir)This is done as static content is not bundled in binary.

The distorless image should consist of both the binary file and static files. Now we can expose the port and then using CMD, run ./main

 EXPOSE 8080
 CMD ["./main"]

CMD [ "executable" ]
from cmd, to build image(we will add tage also) - docker build -t adminnik/go-web-app:v1 . 

The . represents that the command we have ran, the docker file is also in present default dir, if its somehwere else, provide that path.

adminnik is my docker hub username.

# Error 1 

>>> COPY --from=base static /app/static 

while writing final stage, we wrote incorrect syntax for COPY command
It should be like 
COPY --from=base /app/static ./static

# Error 2

docker build -t adminnik/go-web-app:v2
ERROR: "docker buildx build" requires exactly 1 argument.
See 'docker buildx build --help'.

Usage:  docker buildx build [OPTIONS] PATH | URL | -

Start a build

we have to give dir path as well wehre our docker file is located. Correct command is 

docker build -t adminnik/go-web-app:v2 .

! Warning 

In our dockerfile, we are using version 1.21 while in go.mod the developer has mentioned version go 1.22.5. Change version to 1.22.5


After this we will run the image and verify if containerisation is success

  docker run -p 8080:8080 -it adminnik/go-web-app:v1   ( we do port mapping with -p , container port with host port) Inside container the app is running on 8080 port, and we will map it port 8080 on host as well.

  -it - Run the container interactively.

If application is running on local host, it means our containerisation was succesfull.

# Push image to docker hub

docker push adminnik/go-web-app:v2 

This completes our Containerisation where we have used multi stage docker build concept.

## KB YAML Manifest files

We will create a folder name K8S, which will consist of dir manifest. THis manifest consists of deployment.yml. We can also directly deploy this application as a POD, but if we deploy its a POD, we dont get to scale it using replica set, and other features, hence we always go with writing a Deployment file.

It is selector, not selectors. Then it is matchLabels, not matchlabels.

# Why selector definition is required?

Since pod was created with a label, we need to bring that label in our service definition file.
For this, We make use of labels and selectors. Provide a list of labels used to identify the POD. Refer to 
POD definition file used to create the POD. Pull the labels from POD Definition file and place under 
selector section in the service yaml.

Note - The label, which is assigned to pod(which creating pod.yaml), you need to make sure you provide same in selector of service. KB service discovery happens through the labels and selectors.

# Creating service.yaml file

Service: A Kubernetes Service that identifies a set of Pods using label selectors. Unless mentioned otherwise, Services are assumed to have virtual IPs only routable within the cluster network.

Hence, The most imp is selector, the selector must match the label provided in our pod specification in the deployment file.
Target port is container port, the port insdie the container within which our application is running.

# Ingress file

What is Ingress?

    Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

    We have different types of ingress, host and path based, we will be making use of host based ingress. Host based ingress only allows the application traffic on particular hostname which u give.

    Read more about Host vs Path based ingress [here](https://nidhiashtikar.medium.com/kubernetes-ingress-host-based-ingress-and-path-based-ingress-4d82d1eedb14)
    Prerequisites

    You must have an Ingress controller to satisfy an Ingress. Only creating an Ingress resource has no effect.

    You may need to deploy an Ingress controller such as ingress-nginx. You can choose from a number of Ingress controllers.

ingressClassName: nginx - Ingress class name is for the ingress resources to be identified by ingress controller. In some projects there can be multiple ingress controllers, like nginx, AWS ALB, Azure ingress controller etc. So ingress resources within the KB clusters need to be identified by those Ingress controllers. Hence for this purpose, we provide ingressClassName. This ingressClassName tells the nginx ingress controller that this is the ingress resource that you have to watch for. If we are not providing the ingressClassName, then nginx ingress controller will not watch your ingress resource.

Since we are going to implement nginx ingress controller, thats why we have put ingress class name as nginx.

Note - IN ingress yaml, service name and port should be similar to your service yaml


# Setting up EKS 

Go to repo - https://github.com/iam-veeramalla/go-web-app-devops/tree/main/eks

## Prerequisites

kubectl – A command line tool for working with Kubernetes clusters. For more information, see Installing or updating kubectl.

Follow https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/

    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl"
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl.sha256"
    echo "$(cat kubectl.sha256)  kubectl" | shasum -a 256 --check
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
    sudo chown root: /usr/local/bin/kubectl
    kubectl version --client
    rm kubectl.sha256


eksctl – A command line tool for working with EKS clusters that automates many individual tasks. For more information, see Installing or updating.

    brew install weaveworks/tap/eksctl
    eksctl version

AWS CLI – A command line tool for working with AWS services, including Amazon EKS. For more information, see Installing, updating, and uninstalling the AWS CLI in the AWS Command Line Interface User Guide. After installing the AWS CLI, we recommend that you also configure it. For more information, see Quick configuration with aws configure in the AWS Command Line Interface User Guide.

     Create a IAM user, provide access to eks, and Set it up using aws configure
    
# Implementing a EKS cluster using TF

Follow the EKS cluster creation using TF repo.
When you create EKS cluster for the first time, you will have only coredns  kube-system under deployment

# Deployment

Once EKS is created, we take KB manifest, and run the command -

   kubectl apply -f k8s/manifest/deployment.yaml

        O/P
        deployment.apps/go-web-app created

  kubectl get pods 

    go-web-app-9c6685f46-nrhrz   0/1     CrashLoopBackOff   2 (28s ago)   45s

We faced the CrashLoopBackOff error, which we have discussed in deep in docker concept -> CrashLoopBackOfferror
We fixed this by running below commands

    docker build --platform linux/amd64 -t adminnik/go-web-app:3 .
    docker image inspect adminnik/go-web-app:v3 | grep Architecture
    docker push adminnik/go-web-app:v3
    kubectl set image deployment/go-web-app go-web-app=adminnik/go-web-app:v3

    kubectl get pods

# Service and ingress

    amitdhanik@Amits-MacBook-Air go-web-app % kubectl apply -f k8s/manifests/service.yaml   
    service/go-web-app created
    amitdhanik@Amits-MacBook-Air go-web-app % kubectl apply -f k8s/manifests/ingress.yaml 
    ingress.networking.k8s.io/go-web-app created
    amitdhanik@Amits-MacBook-Air go-web-app % 

    o/p

    amitdhanik@Amits-MacBook-Air go-web-app % kubectl get ing
    NAME         CLASS   HOSTS              ADDRESS   PORTS   AGE
    go-web-app   nginx   go-web-app.local             80      9s

    The address is not here.

Deployment, ing, and svc are created.

Right now we cant access our resource directly from the ingress because as seen above, address is not assigned. We need an ingress controller for this so that Ingress controller assigns the address for ingress resource. Once the address is assigned, we can take the IP address, and map it to domain name that we have created in my etc host: go-web-app.local

But to check if our service is working fine or not, we can expose our service in nodeport mode

# Changing to NodePort

So we will edit the type from ClusterIP to NodePort to make it work

kubectl edit svc go-web-app

After this change, we can check if our service is working fine or not even without the ingress configuration. We have exposed the service on NodePort, and we can see if our service can be accessed on the port 30786 on our node ip address

    amitdhanik@Amits-MacBook-Air go-web-app % kubectl get svc                            
    NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    go-web-app   NodePort    172.20.129.160   <none>        80:30786/TCP   2m21s
    kubernetes   ClusterIP   172.20.0.1       <none>        443/TCP        63m
    amitdhanik@Amits-MacBook-Air go-web-app % 

To get our node IP addresses, we can do kubectl get nodes -o wide
    
kubectl get nodes -o wide
NAME                         STATUS   ROLES    AGE   VERSION               INTERNAL-IP   EXTERNAL-IP   OS-IMAGE         KERNEL-VERSION                  CONTAINER-RUNTIME
ip-10-0-1-162.ec2.internal   Ready    <none>   58m   v1.30.2-eks-1552ad0   10.0.1.162    <none>        Amazon Linux 2   5.10.220-209.869.amzn2.x86_64   containerd://1.7.11
ip-10-0-2-148.ec2.internal   Ready    <none>   58m   v1.30.2-eks-1552ad0   10.0.2.148    <none>        Amazon Linux 2   5.10.220-209.869.amzn2.x86_64   containerd://1.7.11

Somehow we were not seeing the EXTERNAL-IP, its because our nodes are not in public subnet. Nodes in public subnets can have public IP addresses assigned to them. Nodes with public IPs (e.g., in a public subnet or if assigned an Elastic IP) will have an EXTERNAL-IP. If EXTERNAL-IP is not present, it usually indicates that the nodes do not have public IPs. This means that when a service is exposed via NodePort, it can be accessed from outside the cluster using these public IP addresses and the NodePort.

EXTERNAL-IP: Empty EXTERNAL-IP generally means nodes are in private subnets or don't have public IPs. Nodes in private subnets usually do not have external IPs unless NAT gateways or other mechanisms are configured to handle external access.

# Why ingress is required

Nodes in private subnets typically do not have public IP addresses. If you try to access services exposed via NodePort, you'll need to use internal IP addresses and ports. External access requires additional configuration, such as setting up a NAT gateway or using an ingress controller with an external load balancer.

# IMPLEMENTING INGRESS CONTROLLER

Till now we have completed containerisation, completed Kubernetes manifest creation, EKS cluster creation, and now we will complete Ingress controller configuration.

https://docs.nginx.com/nginx-ingress-controller/
https://github.com/kubernetes/ingress-nginx

There are 2 Nginx ingress controller - one is provided by nginx,provided by F5, other is community driven nginx ingress controller which is also quite popular.

We will be making use of community driven nginx. You will see the getting started document page. In documentation page, where do you want to install the ingress controller?

We want it on AWS, so we will choose that

In AWS, we use a Network load balancer (NLB) to expose the Ingress-Nginx Controller behind a Service of Type=LoadBalancer.

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml

This will create our ingress controller resource.

    namespace/ingress-nginx created
    serviceaccount/ingress-nginx created

kubectl get pods -n ingress-nginx

    NAME                                    READY   STATUS      RESTARTS   AGE
    ingress-nginx-admission-create-xsgjm    0/1     Completed   0          61s
    ingress-nginx-admission-patch-g6khj     0/1     Completed   0          60s
    ingress-nginx-controller-666487-vv7jm   1/1     Running     0          62s

kubectl edit pod <pod-nam> -n <ns> OR kubectl describe pod <pod-name> -n <ns> -->

HERE YOU Find the ingress controller, in the spec section of pod where ingress-class=nginx mentioned. This is because in ingress.yaml, we have mentioned ingressClassName: nginx, so the ingress controller  will watch (or this pod) will watch for the ingress resources with className as nginx.

    spec:
    containers:
    - args:
        - /nginx-ingress-controller
        - --publish-service=$(POD_NAMESPACE)/ingress-nginx-controller
        - --election-id=ingress-nginx-leader
        - --controller-class=k8s.io/ingress-nginx
        - --ingress-class=nginx

this ingress-class=nginx signifies that in the ingress.yaml, where we have mentioned ingressClassName: nginx, so this pod will watch for the Ingress resources with the class name as nginx.



Just like we use labels and selectors for pods and service discovery, for ingress controller and ingress, this is how ingress class name is used for discovery.

To see if this ingress controller was able to watch our ingress resource

    kubectl get ing

O/P

    amitdhanik@Amits-MacBook-Air go-web-app % kubectl get ing
    NAME         CLASS   HOSTS              ADDRESS                                                                         PORTS   AGE
    go-web-app   nginx   go-web-app.local   a75e27930a8bd40c5afd9424b61f5860-87bed1b105bdb04f.elb.us-east-1.amazonaws.com   80      21m
    amitdhanik@Amits-MacBook-Air go-web-app % 

We can see ingress resource is watched by ingress controller, and it gave us an FQDN

### Conclusion

As we have discussed earlier, I.C will watch for Ingress resource, and creates a LB. So this IC watched for ingress resource, and created a LB on AWS, as per ingress configuration which you can find in load balancer - NLB will be there.

This NLB is created by the ingress controller

for out case, we are telling I.C to create a load balancer, and what that LB has to do, if we access the LB (not on load balancer address) on go-web-app.local, it has to forward the request to the service that we have mentioned below

          service:
            name: go-web-app
            port:
              number: 80

If request is forwarded to the service, the request is then forwarded to the pods.

### Now the LB that is created by IC, so can we access the application on this address?

No, because if we go to our ingress.yaml, we have clearly expalined the load balancer, only accept the request, if someone is accessing the host name -  go-web-app.local

If we were in corporate, we will not put somthing like host: go-web-app.local. Instead we will use amazon.com or flipkart.com, anything that is our organisation host name. We will also be not doing DNS mapping locally, its done differently which is not required here.

# DNS mapping

We have to do a DNS mapping. We will do nslookup <address of Load Balaner>. It gives us the IP . We go to /etc/host, and map this IP with our local hostname 

    amitdhanik@Amits-MacBook-Air go-web-app % nslookup a75e27930a8bd40c5afd9424b61f5860-87bed1b105bdb04f.elb.us-east-1.amazonaws.com
    Server:         2405:201:6810:a0b2::c0a8:1d01
    Address:        2405:201:6810:a0b2::c0a8:1d01#53

    Non-authoritative answer:
    Name:   a75e27930a8bd40c5afd9424b61f5860-87bed1b105bdb04f.elb.us-east-1.amazonaws.com
    Address: 52.7.93.85
    Name:   a75e27930a8bd40c5afd9424b61f5860-87bed1b105bdb04f.elb.us-east-1.amazonaws.com
    Address: 35.153.117.155

    amitdhanik@Amits-MacBook-Air go-web-app % 

sudo vi /etc/hosts
35.153.117.155 go-web-app.local
<IP> go-web-app.local 

Now we can try go-web-app.local/home can now be accessed


