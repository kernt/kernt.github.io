**Controller**:

- **Rewrite Target**: This annotation rewrites the request URL before forwarding it to the service.

nginx.ingress.kubernetes.io/rewrite-target: /

- **SSL Redirect**: To force HTTPS traffic, you can add the following annotation:

nginx.ingress.kubernetes.io/ssl-redirect: "true"

- **Custom Error Pages**: You can specify custom error pages for specific status codes like `404` or `502`:

nginx.ingress.kubernetes.io/custom-http-errors: "404,502"

# Securing Ingress with TLS (HTTPS)

To enable HTTPS, you need to configure **TLS** in your Ingress resource. Kubernetes uses secrets to store TLS certificates.

1. First, create a secret containing the TLS certificate and key:

`kubectl create secret tls tls-secret --cert=path/to/cert.crt --key=path/to/cert.ke`y

2. Modify the Ingress resource to include the TLS configuration:

```
apiVersion: networking.k8s.io/v1 kind: Ingress metadata:   name: example-ingress spec:   tls:   - hosts:     - example.com     secretName: tls-secret   rules:   - host: example.com     http:       paths:       - path: /app1         pathType: Prefix         backend:           service:             name: app1-service             port:               number: 80
```

With this configuration, traffic to `https://example.com/app1` will be routed to `app1-service`, and the Ingress Controller will terminate SSL using the provided certificate.

https://medium.com/@Shamimw/kubernetes-a-complete-tutorial-part6-services-loadbalancer-ingress-9f657bb100c5
