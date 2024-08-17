
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


# Helm Chart configuration

# Concept

Helm is used When we want to deploy our application to diff envs. Consider below eg ->

We have our deployment, service, and ingress.yaml, and all these are hardcoded. Lets look at our image - image: adminnik/go-web-app:v3 which we are using in deployment.yaml. If we want something for dev, prod, we would need to make separate folders and files for each like k8s/manifest/dev -> image: adminnik/go-web-app:dev, k8s/manifest/prod ->image: adminnik/go-web-app:prod

So instead of doing this, we can use helm , and in helm chart, we can variablise these kinds of things.

So to our testing/dev team, we can jsut tell them that pass the tag name as variable.


# Steps

1. We will create a folder name helm
2. Install helm locally

    brew install helm
    helm version

3. Switch to helm dir, and run the command

    cd helm
    helm create go-web-app-chart

4. You can see helm has already created some files for us

amitdhanik@Amits-MacBook-Air helm % cd go-web-app-chart 
amitdhanik@Amits-MacBook-Air go-web-app-chart % ls 
Chart.yaml charts          templates       values.yaml

If you know what is values.yaml, wht is templates, and what is chart.yaml, you know helm


# Chart.yml 

Chart provides information about the chart. We can assume it as metadata. For eg, type: application
Initially chart-version will start with version 0.1.0

# Templates

When we create helm dir structure like this ( with command helm create go-web-app-chart), remove everything that is there inside this template

  rm -rf *

And now inside the templates, just copy your KB manifests (for us we have deployment.yaml, service.yaml & ingress.yaml)

And now we can variablise our deployment.yaml

vi deployment.yaml

Change image tag to a Jinja 2 templating followed by .Values.image.tag

   image: adminnik/go-web-app:{{ .Values.image.tag }}

This means, that helm, whenever executed will look for the tag from values.yaml file

# Values.yaml

Remove everything from your values.yaml, and copy the values.yaml that is there in go-web-app-devops repo. Here we are using .Values.image.tag

For our reference, we can variablise other things as well, such as repository name, ingress configuration.

So now when you run your deployment file, it will go to values.yaml, and go to image.tag -->"10016307834"
Now this stringe would be populated dynamically evry time CI/CD is run. we will update values.yaml of helm with latest image that we created in CI, and using ArgoCD, that latest image with latest tag will automatically be deployed

Rigt now our tag is v3, so we will use that in place of 10016307834

replicaCount: 1

image:
  repository: abhishekf5/go-web-app
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  #tag: "10016307834"
  tag: "v1"

ingress:
  enabled: false
  className: ""
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific


Now we will install everything through helm, so we will delete everything

  kubectl delete deploy go-web-app
  kubectl delete svc go-web-app
  kubectl delete ing go-web-app

  kubectl get all (resulte in nothing)

O/P

      amitdhanik@Amits-MacBook-Air go-web-app-chart % kubectl get all
      NAME                              READY   STATUS    RESTARTS   AGE
      pod/go-web-app-6654f6598f-fmt7p   1/1     Running   0          28m

      NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
      service/go-web-app   ClusterIP   172.20.88.172   <none>        80/TCP    27m
      service/kubernetes   ClusterIP   172.20.0.1      <none>        443/TCP   47m

      NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
      deployment.apps/go-web-app   1/1     1            1           28m

      NAME                                    DESIRED   CURRENT   READY   AGE
      replicaset.apps/go-web-app-6654f6598f   1         1         1       28m
      amitdhanik@Amits-MacBook-Air go-web-app-chart % kubectl delete deploy go-web-app
      deployment.apps "go-web-app" deleted
      amitdhanik@Amits-MacBook-Air go-web-app-chart % kubectl get deploy 
      No resources found in default namespace.
      amitdhanik@Amits-MacBook-Air go-web-app-chart % kubectl get svc
      NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
      go-web-app   ClusterIP   172.20.88.172   <none>        80/TCP    27m
      kubernetes   ClusterIP   172.20.0.1      <none>        443/TCP   48m
      amitdhanik@Amits-MacBook-Air go-web-app-chart % kubectl delete svc go-web-app
      service "go-web-app" deleted
      amitdhanik@Amits-MacBook-Air go-web-app-chart % kubectl get svc              
      NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
      kubernetes   ClusterIP   172.20.0.1   <none>        443/TCP   48m
      amitdhanik@Amits-MacBook-Air go-web-app-chart % kubectl get ing   
      NAME         CLASS   HOSTS              ADDRESS                                                                         PORTS   AGE
      go-web-app   nginx   go-web-app.local   a1944ccdec71d49d2af10c6252b6f005-f6bff86f3bac8e03.elb.us-east-1.amazonaws.com   80      26m
      amitdhanik@Amits-MacBook-Air go-web-app-chart % kubectl delete ing go-web-app
      ingress.networking.k8s.io "go-web-app" deleted
      amitdhanik@Amits-MacBook-Air go-web-app-chart % kubectl get all              
      NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
      service/kubernetes   ClusterIP   172.20.0.1   <none>        443/TCP   49m
      amitdhanik@Amits-MacBook-Air go-web-app-chart % 

Since we created this chart before - helm create go-web-app-chart

this dir consist of all our manifest file, insdie templates. and the values.yaml consist of image details

Now we can install everything with helm - helm install go-web-app ./go-web-app-chart

Now after this, if you do kubectl get deployment, you can see your pods up and running.
Similarly for kubectl get svc, kubectl get ing

If you do kubectl edit deploy go-web-app, you can see that in image, you will get the v1 tag, which has been take from values.yaml. It replaced the tag in deployment.yaml

      amitdhanik@Amits-MacBook-Air helm % helm install go-web-app ./go-web-app-chart 
      NAME: go-web-app
      LAST DEPLOYED: Sat Aug 17 17:43:29 2024
      NAMESPACE: default
      STATUS: deployed
      REVISION: 1
      TEST SUITE: None
      amitdhanik@Amits-MacBook-Air helm % kubectl get deployment
      NAME         READY   UP-TO-DATE   AVAILABLE   AGE
      go-web-app   1/1     1            1           14s
      amitdhanik@Amits-MacBook-Air helm % kubectl get svc
      NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
      go-web-app   ClusterIP   172.20.207.39   <none>        80/TCP    39s
      kubernetes   ClusterIP   172.20.0.1      <none>        443/TCP   55m
      amitdhanik@Amits-MacBook-Air helm % kubectl get ing
      NAME         CLASS   HOSTS              ADDRESS                                                                         PORTS   AGE
      go-web-app   nginx   go-web-app.local   a1944ccdec71d49d2af10c6252b6f005-f6bff86f3bac8e03.elb.us-east-1.amazonaws.com   80      44s
      amitdhanik@Amits-MacBook-Air helm % 

To uninstall everything, we can run - helm uninstall go-web-app


# Summary

We have completed Helm, EKS with TF, Containerisation with Docker and Ingress controller

### CI/CD

Whenever developer comits a change or creates a pull request, the CI/CD pipeline is triggered, where as part of CI, using Github actions, we will run multiple stages. These stages are explained in detail in below -->


# Implementing CI using GITHUB ACTIONS

We will be developing our CI whenever a commit happens
In CI, we will implement mulitple stages. The  stages would be implemented in below order - 

1. Build and Test (unit test)
2. We will run the static code analysis.
3. We will create Docker image, and push the docker image
4. Update helm values.yaml (automatically updated) with docker image tag that we have created.

# CD using GITOPS - ARGO CD

After this CD comes in picture, where we will be making use of Argo CD. ArgoCD watches the helm chart. 

Once helm tag is updated, and values.yaml is updated,  ArgoCD will pull the Helm chart, (if helm chart already exists, it will update the helm chart) so a new version is deployed on KB cluster.

It is important to understand how CI and CD are connected. We can add n no of stages in CI, its not difficult, whats difficult is how CI every new commit is updated to your helm, and how CD immediately picks up that change, and deploys that change on KB cluster.

### IMPLEMENTING GITHUB CI PIPELINE

To implement a github CI pipeline

1. Create a dir as .github
2. Inside the .github dir, create another folder as workflows
3. Inside workflows, you can create a file, eg ci.yaml

# Workflows

A workflow is a configurable automated process that will run one or more jobs. Workflows are defined by a YAML file checked in to your repository and will run when triggered by an event in your repository, or they can be triggered manually, or at a defined schedule.

A repository can have multiple workflows, each which can perform a different set of tasks such as:

  Building and testing pull requests.
  Deploying your application every time a release is created.
  Adding a label whenever a new issue is opened.


We need to provide a trigger for github so github knows when to build the pipeline. This is called workflow in Github, which is similar to build trigger in Jenkins.

# push condtions

You have to mention clearly when github has to trigger the workflow. We will build the pipeline whenever someone pushes to branch main. Also ignore some paths, for eg, readme.md (if someone is updating documentation, we dont have to run the CI pipeline there). If someone is updating anything inside the helm folder, we dont have to run the CI pipeline. So we can mention all these.

name: Go

on: 
  push:
    branches:
      - main
    paths-ignore:
      - 'README.md'
      - 'helm/**'

# Jobs

In Github actions, inside the Jobs, we will start writing the stages. N no of stages can be mentioned inside this jobs section.

**Build and Test Job**

We have 3 stages defined in our job sectiion, all stages doing a specific set of tasks.

  Build
  Code qualtiy
  push
  update-newtag-in-helm-chart

Lets discuss them in detail

Our first job is going to be the build and test job. YOu can find the below syntax in github actions docs in section - Use cases and examples (Build and test).

1. When we run a job, we need to provide the vm or container where github actions will run this particular stage. We are saying github to use its own runner, we can also use our runner as well.

2. Checkout the source code
3. Install go lang in it(616-617)
4. Run the build command (622-623)
5. Test the unit test cases(624-625)

2. The next stage is code quality

1. For every stage that we write, we need to provide a image(code-quality here) as github action need to execute this on the runner, so it needs to know on which runner it needs to run.

2. The steps mentioned from 641-645 are used everywhere in a golang project for static code analysis. Golangci is used for linting. You can get all the actions specified pipeline on **GITHUB ACTIONS MARKETPLACE**
We have actions defined for most of this things, and we dont have to specifiy actions on our own, unless and until it is not available in action marketplace

3. We will push docker image to docker hub

When we do docker login, Since its not good to push secrets in a CI pipeline, we can go to our Repo, click on settings , Secrets and variables, click on Actions - Click on repository secrets - New repo secrets. Provide the dockerhub username, and dockerhub token. Generate token from dockerhub

tags: ${{ secrets.DOCKERHUB_USERNAME }}/go-web-app:${{github.run_id}}

Every commit creates a new docker image, and tag for new docker image that gets created should is github.run_id
line 56 to 62 
    - name: Build and Push action
      uses: docker/build-push-action@v6
      with:
        context: .
        file: /Dockerfile
        push: true
        tags: ${{ secrets.DOCKERHUB_USERNAME }}/go-web-app:${{github.run_id}}

If 100 images are created, they are all pushed to github repo with a new {{github.run_id}}

3. Last stage is to update helm chart with new tag that CI has created.

We have used needs: push here

It means only after the stage push is complted, it will trigger the last stage.

we have created a secret token as well, because for pushing to github repo, we need github token. Create secret within same repo where u created docker token. share the personal access token from developer setting (classic tokens), give permission and generate token.

Once we have updated the values.yaml using sed command, we have to push the commit again. so we will add and push the changes again

name: Go

on: 
  push:
    branches:
      - main
    paths-ignore:
      - 'README.md'
      - 'helm/**'

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - name: Checkout repo
      uses: actions/checkout@v4
    - name: Set up Go 1.22
      uses: actions/setup-go@v2
      with:
        go-version: 1.22
    - name: Build
      run: go build -o go-web-app
    - name: Test
      run: go test ./..
  
  code-quality:
    runs-on: ubuntu-latest

    steps: 
    - name: Checkout repository
      uses: actions/checkout@v4
    
    - name: Run golangci-lint
      uses: golangci/golangci-lint-action@v6
      with:
        version: v1.56.2

Commit the changes to github

# ArgoCD

Everytime the CI pipeline is run, Argo CD has to identify the change, and push it to KB cluster.

# Implementing ARGOCD

Firstly we have to install ARGO CD

Install Argo CD using manifests

  kubectl create namespace argocd
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Access the Argo CD UI (Loadbalancer service)
  kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

Get the Loadbalancer service IP

  kubectl get svc argocd-server -n argocd


Once the lB is deployed, you can access the ArgoCD. To login, you need password, which can be retrieved by - 

  kubectl get secrets -n argocd
    select the admin-secret one, and run

  kubectl edit secret ***-admin-secret -n argocd

  copy the password, run 
  
  echo <encoded-password> | base64 --decode  
  Copy till % (excluding %)

Our ArgoCD is on the same KB cluster


Click on New app - 