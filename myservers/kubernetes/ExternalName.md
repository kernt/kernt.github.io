## ExternalName

The **ExternalName** Service is a special case that maps a Service to an external DNS name. Instead of creating a proxy to route traffic, this Service type simply returns a CNAME record that points to an external resource.

- **Use case**: When you need to point to external services (e.g., an external database or an API) by a DNS name rather than managing network routing within the cluster.
- **Example**: Accessing external services such as SaaS products or databases hosted outside the cluster.

```sh
apiVersion: v1  
kind: Service  
metadata:  
  name: my-external-service  
spec:  
  type: ExternalName  
  externalName: example.com
```

