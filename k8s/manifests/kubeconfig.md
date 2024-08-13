

By default, kubectl uses the kubeconfig file located at ~/.kube/config.

After we had created our KB cluster on EKS using tf, we were getting the following errors while creating deployment -->

amitdhanik@Amits-MacBook-Air go-web-app % kubectl apply -f k8s/manifests/deployment.yaml 

    error: error validating "k8s/manifests/deployment.yaml": error validating data: failed to download openapi: Get "https://127.0.0.1:64637/openapi/v2?timeout=32s": dial tcp 127.0.0.1:64637: connect: connection refused; if you choose to ignore these errors, turn validation off with --validate=false

The main reason for this was that we had earlier created a kind KB cluster on our local machine, and our  current kubeconfig file and its configuration were pointing to that cluster

To see the current context that kubectl is using, run below and this will display the name of the current context, which is a set of configuration settings including cluster, user, and namespace.

    amitdhanik@Amits-MacBook-Air go-web-app % kubectl config current-context

    kind-devops-with-amit

To view all contexts defined in your kubeconfig file, use:
    amitdhanik@Amits-MacBook-Air go-web-app % kubectl config get-contexts

    CURRENT   NAME                    CLUSTER                 AUTHINFO                NAMESPACE
    *         kind-devops-with-amit   kind-devops-with-amit   kind-devops-with-amit 

To see detailed information about the current context, use:

    amitdhanik@Amits-MacBook-Air go-web-app % kubectl config view --minify

    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: DATA+OMITTED
        server: https://127.0.0.1:64637
    name: kind-devops-with-amit
    contexts:
    - context:
        cluster: kind-devops-with-amit
        user: kind-devops-with-amit
    name: kind-devops-with-amit
    current-context: kind-devops-with-amit
    kind: Config
    preferences: {}
    users:
    - name: kind-devops-with-amit
    user:
        client-certificate-data: DATA+OMITTED
        client-key-data: DATA+OMITTED
    amitdhanik@Amits-MacBook-Air go-web-app % 

To see the entire content of your kubeconfig file, run: kubectl config view


# How do we update our kubeconfig to use the EKS cluster that we have created on AWS?



### 1. **Ensure EKS Cluster Connectivity**

First, you need to ensure that your local `kubectl` command is configured to interact with your EKS cluster. This typically involves a few steps:

1. **Update kubeconfig**: Use the AWS CLI to update your local kubeconfig file to include the context for your EKS cluster. This allows `kubectl` to communicate with the EKS cluster.

   ```bash
   aws eks update-kubeconfig --region <your-region> --name <your-cluster-name>
   ```

   - Replace `<your-region>` with the AWS region where your EKS cluster is running.
   - Replace `<your-cluster-name>` with the name of your EKS cluster.

    aws eks update-kubeconfig --region us-east-1 --name amit-eksafNNSbyU

O/P

amitdhanik@Amits-MacBook-Air go-web-app % aws eks update-kubeconfig --region us-east-1 --name amit-eksafNNSbyU
Added new context arn:aws:eks:us-east-1:975050301329:cluster/amit-eksafNNSbyU to /Users/amitdhanik/.kube/config
amitdhanik@Amits-MacBook-Air go-web-app % 


2. **Verify Connection**: Ensure you can connect to the cluster by running:

   ```bash
   kubectl get nodes
   ```

   This command should list the nodes in your EKS cluster if the connection is successful.

amitdhanik@Amits-MacBook-Air go-web-app % kubectl get nodes
E0811 17:19:43.980706   72257 memcache.go:265] couldn't get current server API group list: Get "https://BAE15D76D41E29FFF2CFE01C38C54140.sk1.us-east-1.eks.amazonaws.com/api?timeout=32s": dial tcp 10.0.2.13:443: i/o timeout
E0811 17:20:13.985872   72257 memcache.go:265] couldn't get current server API group list: Get "https://BAE15D76D41E29FFF2CFE01C38C54140.sk1.us-east-1.eks.amazonaws.com/api?timeout=32s": dial tcp 10.0.2.13:443: i/o timeout

We got this error which we needed to solve.

We delete the old config file with kind cluster - rm ~/.kube/config
and re updated kube config - aws eks update-kubeconfig --region us-east-1 --name amit-eksafNNSbyU

### 2. **Deploy Your Kubernetes Manifests**

Once your `kubectl` is properly configured to connect to your EKS cluster, you can apply your Kubernetes manifests. Here’s what happens when you apply your manifests:

1. **Deployment Creation**: When you run:

   ```bash
   kubectl apply -f k8s/manifests/deployment.yaml
   ```

   - **Deployment**: The `deployment.yaml` manifest will instruct Kubernetes to create or update a deployment in the EKS cluster. This deployment specifies how many replicas of a pod should run, what container image to use, and other settings.
   - **Pods**: The deployment will create the specified number of pods with the container image and configurations defined in the manifest.

2. **Service Creation**: If you then apply:

   ```bash
   kubectl apply -f k8s/manifests/services.yaml
   ```

   - **Service**: The `services.yaml` manifest will create a service that exposes your pods. Depending on the type of service (ClusterIP, NodePort, LoadBalancer), it will define how your application can be accessed within the cluster or externally.

3. **Ingress Creation**: If you apply:

   ```bash
   kubectl apply -f k8s/manifests/ingress.yaml
   ```

   - **Ingress**: The `ingress.yaml` manifest will create an ingress resource that defines rules for routing external HTTP(S) traffic to your services within the cluster. You need an ingress controller (like AWS ALB Ingress Controller) running in your cluster to process ingress rules.

### 3. **Connecting Everything**

Here’s how everything connects:

1. **Pod Deployment**: The deployment creates pods running your Docker container. Each pod gets a unique IP address within the Kubernetes cluster.

2. **Service Exposure**: The service exposes these pods within the cluster or externally, depending on the service type. For example:
   - **ClusterIP**: Only accessible within the cluster.
   - **NodePort**: Accessible on `<NodeIP>:<NodePort>`.
   - **LoadBalancer**: Creates an AWS ELB and provides an external IP address or DNS name.

3. **Ingress Routing**: If using an ingress, it provides external access to your services based on rules (e.g., hostnames or paths). The ingress controller manages traffic routing to the appropriate services.

### 4. **Check Deployment**

After applying your manifests, you can verify the deployment status with:

```bash
kubectl get deployments
kubectl get pods
kubectl get services
kubectl get ingress
```

These commands will show the status of your deployments, pods, services, and ingress resources.

### Summary

1. **Configure `kubectl`**: Ensure your `kubectl` is configured to connect to your EKS cluster.
2. **Apply Manifests**: Use `kubectl apply` to create deployments, services, and ingress rules.
3. **Verify**: Check the status of your deployments and resources using `kubectl get` commands.

By following these steps, you ensure that your Kubernetes manifests are applied to your EKS cluster, and your application is properly deployed and exposed.