

# Helm commands

# Node commands

    kubectl describe node <node-name>   - to check node details and configurations.


# Deployment commands

  kubectl get all
  kubectl get deployments
  kubectl describe deployment <deployment-name>
  kubectl set image deployment/go-web-app go-web-app=adminnik/go-web-app:v3 (if you have changed the docker image)
  kubectl delete deploy go-web-app - delete deployment

# Replica Set

  kubectl get replicasets
  kubectl describe replicaset <replicaset-name>

# Debug commands

    kubectl logs <pod-name>
    kubectl logs <pod-name> -c <container-name> (If the pod has multiple containers, specify the container name:)
    kubectl describe pod <pod-name>

# Node architecture

    kubectl describe node <node-name>  - Ensure that the nodes in your EKS cluster are of the correct architecture. You can check the instance type and its architecture.For most standard AWS EC2 instances, the architecture should be x86_64.


# kube-config commands 

Located at  ~/.kube/config


    kubectl config current-context
    kubectl config get-contexts
    kubectl config view --minify
    
    aws eks update-kubeconfig --region us-east-1 --name amit-eksw0yRQTCI (the server should be pointing to correct endpoint.)
    aws eks describe-cluster --name amit-eksafNNSbyU --region us-east-1

Understanding the aws-auth ConfigMap
The aws-auth ConfigMap is a critical configuration in EKS that defines which IAM users and roles have permissions to access Kubernetes resources in the cluster. Even if you have the appropriate IAM policies attached to your user or role, you need to explicitly grant access in the aws-auth ConfigMap for your IAM entities to interact with Kubernetes resources.

    kubectl get configmap aws-auth -n kube-system -o yaml


# Network connectivity 

curl -v  https://BAE15D76D41E29FFF2CFE01C38C54140.sk1.us-east-1.eks.amazonaws.com/api

# Service
    kubectl get service go-web-app
    kubectl edit svc go-web-app
    kubectl get nodes -o wide
    kubectl get pods
    kubectl get ing
    kubectl describe service <service-name> - to see the type of service and its assigned IPs or load balancer information.

# Create or Update resources from a file
    kubectl apply -f k8s/manifests/deployment.yaml
    kubectl apply -f k8s/manifests/service.yaml
    kubectl apply -f k8s/manifests/ingress.yaml

# Ingress service
    kubectl get ing
