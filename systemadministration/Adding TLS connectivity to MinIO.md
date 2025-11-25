In a previous article I introduced MinIO as an S3 compatible object storage system. For experimental purposes, we deployed this using HTTP connections. In this article I add TLS connections to ensure all traffic into and out of the MinIO service is encrypted.
# Set up

To add TLS connections to a server, you need to install a private key and a public certificate which is signed by a public, well-known Certificate Authority (CA).

When a client connects to your server, it will then check the certificate against the CA.

In this article we will use a wildcard Let’s Encrypt certificate to secure a MinIO object storage solution for Kubernetes.

Rather than copying everything from those articles, I am going to assume you have followed both:

1. [Object Storage in your Kubernetes cluster using MinIO](https://medium.com/@martin.hodges/object-storage-in-your-kubernetes-cluster-using-minio-ad838decd9ce)
2. [Adding a wildcard Let’s Encrypt certificate to your server without a web server](https://medium.com/@martin.hodges/adding-a-wildcard-lets-encrypt-certificate-to-your-server-without-a-web-server-2e86e4e292ab)

If you have followed these articles, you will now have a MinIO server on a standalone server and have both `fullchain.pem` and `privkey.pem` files.
# Configuring TLS on MinIO

Log in to your MinIO server. As `root`, carryout the following changes.
## Install the certificates

First you need to install the Let’s Encrypt certificates. As they are text files you can transfer them by using cut and paste to copy the text. Make sure you get the header and footer rows and include all the hyphens.

First create the path to hold the certificates:

`mkdir -p /etc/minio/certs`

If you have followed the previous articles, this path will exist and will have two self-signed certificates:

- `private.key`
- `public.crt`

We will be replacing these files with the public certificates. You may want to copy them to an archive folder.

Replace the text in these two files with the contents of the two public files:

- `fullchain.pem` -> `public.crt`
- `privkey.pem` -> `private.key`
## Update MinIO configuration

Edit the following file (remember to replace the < > fields with your values):

`**/etc/default/minio**`

```sh
# define where the data is to be held  
MINIO_VOLUMES="/mnt/minio"  
  
# define where to find the TLS certificates  
MINIO_OPTS="--certs-dir /etc/minio/certs --console-address :9001"  
  
# define the admin user (min 3 characters)  
MINIO_ROOT_USER=<minio admin user>  
  
# define the defaut admin user password (min 8 characters)  
MINIO_ROOT_PASSWORD=<minio admin password>
```

Make sure the `MINIO_OPTS` are uncommented.
Now restart the service:

```sh
sudo systemctl restart minio  
sudo systemctl status minio
```
# Test the service

To test the service you will need to create an URL that maps to the IP address of your MinIO service.

If, like me, you have the MinIO service in your Virtual Private Cloud with a connection via OpenVPN, then you will need to an entry to the hosts file on your local machine, to include a host name that is compatible with the wildcard Let’s Encrypt certificate you created. For me this means I added the following line:

`**/etc/hosts**`

`10.240.0.166 minio.requillion-solutions.cloud`

Now, when I went to `https://minio.requillion-solutions.cloud:9001` in my browser, I was presented with my MinIO console login page.

After logging in, I could check in the network tab of my Chrome browser (`Inspect` right-click menu option), that all my calls to the MinIO API were also over HTTPS.

Note that you may have to restart other services that have been configured to access the service over HTTP.
# Accessing MinIO as a custom host

From the test above we can access MinIO through its console over TLS. We needed to add the hostname to our `hosts` file so that we can resolve the custom host (in my case `minio.requillion-solutions.cloud`) to the private IP address on our VPC.

This works fine for the console but what services in our Kubernetes cluster to access MinIO as an external service?

Whilst it may be tempting to add an `ExternalService` to the cluster, this will not work as the IP address will not work with the certificate. We need to be able to access the service via its custom host name.

You can add custom host names to a Pod’s /etc/host file using the `hostAliases` entry as part of the Pod’s `spec`. This does not cover all use cases and it is better to install the custom host into the `coreDNS` system that is installed by default when your Kubernetes cluster is created.

To do this, you first need to edit the `coreDNS` configuration. You can do this with this command:

 `kubectl -n kube-system edit configmap/coredns`

This will open up a `vi` editor (you may need to Google how to use `vi`) with the `configMap` for the `coreDNS` pod.

You can now add a hosts section to this file. Under the line `loadbalance`, add the following:

```sh
hosts <filename> <zone> {  
    <ip address> <custom hostname>  
    fallthrough  
}
```

Ok, so there are a lot of < > fields here. I’ll add a bit of explanation first.

- `filename` — this is the filename that `coreDNS` will use to create its internal configuration (I would use `<domain>.hosts`)
- `zone` — this is the zone that this section will manage and is effectively the domain we are talking about
- `ip address` — this is the IP address of the MinIO server
- `custom hostname` — this is the Fully Qualified Domain Name (FQDN) for the MinIO server

May be that is not quite clear enough? Let’s give you my example. I am using the custom host name of `minio.requillion-solutions.cloud`.

I insert the following for my custom host definition:

```c
        hosts requillion-solutions.hosts requillion-solutions.cloud {  
                10.240.0.165 minio.requillion-solutions.cloud  
                fallthrough  
        }
```

Once you have added this section you can save with `esc`, `:wq`

Once saved, `coreDNS` should pick up your changes within 5 seconds. If not, you may need to delete all the `coreDNS` Pods, which you can find with:

`kubectl get pods -A | grep coreDNS`

Your Pods should now be able to access the MinIO service by using its custom host name.