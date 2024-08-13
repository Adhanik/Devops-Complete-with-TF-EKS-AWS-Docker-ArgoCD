

The instructor had created eks cluster using below command - eksctl create cluster --name demo-cluster --region us-east-1
does above created nodes in public subnet?

Then he created Deployment, ingress, and svc using command kubectl apply -f k8s/manifests/service.yaml   , kubectl apply -f k8s/manifests/ingress.yaml and kubectl apply -f k8s/manifest/deployment.yaml

But I had created EKS with TF, and my nodes are in private subnet. 

After that  Then he runs below command - 

kubectl get ing
    NAME         CLASS   HOSTS                      ADDRESS   PORTS   AGE
    go-web-app   nginx   go-web-app.local             80      9s

    He tells the The address is not here.


the explanation he gave for this is - >

Right now we can access our resource directly from the ingress because as seen above, address is not assigned. We need an ingress controller for this so that Ingress controller assigns the address for ingress resource. Once the address is assigned, we can take the IP address, and map it to domain name that we have created in my etc host: go-web-app.local


could this be because his nodes get created on public subnet?

# Sol
Yes, the behavior you're observing can indeed be related to whether the nodes are in public or private subnets. Here's a breakdown of how this affects your setup:

### Public vs. Private Subnets

1. **Nodes in Public Subnets**:
   - Nodes in public subnets can have public IP addresses assigned to them. This means that when a service is exposed via NodePort, it can be accessed from outside the cluster using these public IP addresses and the NodePort.

2. **Nodes in Private Subnets**:
   - Nodes in private subnets typically do not have public IP addresses. If you try to access services exposed via NodePort, you'll need to use internal IP addresses and ports. External access requires additional configuration, such as setting up a NAT gateway or using an ingress controller with an external load balancer.

### Ingress Controller and LoadBalancer Services

- **Ingress Controller**: 
  - An Ingress Controller manages ingress resources and typically provisions a load balancer to provide external access to the services behind it. If the `ADDRESS` field in `kubectl get ing` is empty, it usually means that the ingress controller has not been set up or is not yet fully configured.

- **LoadBalancer Service**:
  - For external access to services directly, you can use a `LoadBalancer` service type. This will provision a load balancer with an external IP address, making your service accessible from outside the cluster. The `LoadBalancer` service works well with public subnets but requires proper setup if nodes are in private subnets.

### Explanation of Commands and Observations

1. **`eksctl create cluster`**:
   - By default, `eksctl` creates nodes in public subnets if you don't specify otherwise. Public subnets are configured to have direct access to the internet.

2. **Changing Service Type to NodePort**:
   - When you change the service type to NodePort, you expose the service on a port on every node’s IP address. For nodes in a public subnet, this means you can access the service via the public IP address of any node and the NodePort.
   - If the nodes are in private subnets, you can only access the NodePort service from within the VPC or via a NAT gateway if it's set up for external access.

3. **Viewing EXTERNAL-IP**:
   - When using NodePort, if nodes are in a public subnet, you can see the `EXTERNAL-IP` because these nodes are reachable from the internet.
   - For private subnets, the `EXTERNAL-IP` field may not be populated because nodes do not have public IPs.

### Solutions and Next Steps

1. **Check Node Configuration**:
   - Verify the subnet configuration of your EKS nodes. If nodes are in private subnets and you need external access, you might want to configure a NAT gateway or use a load balancer.

2. **Set Up an Ingress Controller**:
   - Install an ingress controller (e.g., NGINX) that will handle ingress resources and provide external access to services. This typically involves setting up a `LoadBalancer` service for the ingress controller, which will then manage the routing of external traffic to your services.

3. **Update Service Configuration**:
   - For services needing external access, consider changing the service type to `LoadBalancer` if you're using public subnets or configuring a NAT gateway if using private subnets.

4. **DNS and Host Mapping**:
   - Once you have an external IP or a domain name from the ingress controller, update your `/etc/hosts` file or DNS settings to map your domain to the provided IP.

### Summary

- **Nodes in Public Subnets**: Show `EXTERNAL-IP` and allow direct NodePort access.
- **Nodes in Private Subnets**: Do not show `EXTERNAL-IP` and require additional configuration for external access.
- **Ingress Controller**: Necessary for managing ingress resources and providing external access through a load balancer.

By understanding these configurations, you can better manage how services are exposed and accessed in your Kubernetes cluster.