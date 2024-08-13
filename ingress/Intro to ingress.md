
Read this article here - https://nidhiashtikar.medium.com/kubernetes-ingress-host-based-ingress-and-path-based-ingress-4d82d1eedb14

Kubernetes Ingress provides various types of routing strategies to manage incoming traffic within a cluster. Here are some common types of Kubernetes Ingress:
    Host-Based Ingress
    Path-Based Ingress
    Simple Fanout
    Name-Based Virtual Hosting
    TLS Termination
    Redirects and Rewrites

Kubernetes Ingress can be classified into different types based on how they manage and route incoming traffic. Today we are going to see Host-Based Ingress and Path-Based Ingress.

### Host-Based Ingress:
Host-based Ingress routes incoming traffic based on the host header of the HTTP request. With host-based Ingress, you can configure different routing rules for different domain names.

For example: you can route requests for example.com to one set of services and requests for subdomain.example.com to another set of services.

Use Case: Hosting multiple websites or services on the same cluster, each with its own domain or subdomain.

Let’s say you have two web applications, app1 and app2, each with its own domain:
    app1 is accessible via app1.example.com.
    app2 is accessible via app2.example.com.

#Here's an example of a host-based Kubernetes Ingress resource YAML


apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-based-ingress
spec:
  rules:
    - host: app1.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app1-service
                port:
                  number: 80
    - host: app2.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app2-service
                port:
                  number: 80

#In this example:
#Requests to app1.example.com are routed to the app1-service
#Requests to app2.example.com are routed to the app2-service

### Path-Based Ingress:

Path-based Ingress routes incoming traffic based on the path of the HTTP request URL. With path-based Ingress, you can define different routing rules for different URL paths within the same domain.

For example: you can route requests to /app1 to one set of services and requests to /app2 to another set of services, all under the same domain.

Use Case: Hosting multiple applications or API endpoints on the same domain.
Let’s consider a scenario where you have a single domain, example.com, hosting two different applications:
    Requests to /app1 should be routed to app1.
    Requests to /app2 should be routed to app2.

#Here's an example of a path-based Kubernetes Ingress resource YAML


apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-ingress
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /app1
            pathType: Prefix
            backend:
              service:
                name: app1-service
                port:
                  number: 80
          - path: /app2
            pathType: Prefix
            backend:
              service:
                name: app2-service
                port:
                  number: 80

#In this example:
#Requests to /app1 under example.com are routed to the app1-service
#Requests to /app2 under example.com are routed to the app2-service
You can use Kubernetes Ingress to route traffic based on either the requested host (domain) or path, providing flexibility in how you expose and manage your applications within a Kubernetes cluster.