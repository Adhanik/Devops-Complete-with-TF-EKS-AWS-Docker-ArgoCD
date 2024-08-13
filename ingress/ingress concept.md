
THere are 3 things to remember wrt to ingress.

1. Ingress
2. Ingress Cotroller
3. Load Balancer

Ingress conroller will watch the Ingress resource and creates the LoaD BALANCER

In a KB cluster, you cannot create Load Balancer by yourself. It is not easy to configure LB every time. Hence we write a Ingress file, Ingress controller(written in go), written by LB companies, in our case which is nginx. Similarly AWS LB is written by AWS, all these are ingress controller, which is basically a GO program, that is written by LB company.

# What does ingress controller do?

It watch for the ingress resource and creates LB as per the ingress configuration.

If we look at our ingress configuration, we are saying to create a LB. What this LB has to do? If i access the LB, not on LB address, but on go-web-app.local, it has to forward the request to the particular service which we have mentioned in service section. If request is forwaded to service, it will forward the request to the PODS.

We are using a host based ingress, read more about it in Intro to ingress.

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: go-web-app
spec:
  ingressClassName: nginx
  rules:
  - host: go-web-app.local
    http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: go-web-app
            port:
              number: 80

