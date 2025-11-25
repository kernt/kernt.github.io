recently came across the need to use S3 compatible storage within my Kubernetes cluster but, given I want a self-contained cluster, I did not want to go to AWS. The alternative is MinIO. In this article I describe how I have created a MinIO service for use with my cluster.

# What is MinIO?

MinIO is an Object Storage solution. Ok, so that might not mean much to you so let’s just look at Object Storage first. I am assuming you are familiar with filesystem storage.

![1_qyJoBsb12XoKoTKziyG8Cw.webp]

In a file system, such as the one on your laptop, you store your files using a filename and you store them at a path. The filesystem keeps track of the directory that determines where each file is on the underlying disks.

This form of storage has been around for a long time and is most familiarly. Back in the late 1990s people started thinking that storage of information would not meet the needs of the Internet. These reasons included:

- Limited metadata about the files
- No redundancy (loose disks and you may lose data)
- File structures become unmanageably deep
- Difficulties in defining where files should go or where to find them
- Limited data storage size

What the world needed was a way of taking a blob of data (think of that as a file), with all sorts of user defined metadata and having it stored in a way that it could easily be accessed. Not only that but stored in a way that is secure, available, unlimited (writes and/or reads) and highly, highly reliable.

Object storage was born.

![1_FuyRt5yNzb6h35CuY25tbg.webp]

With object storage, the data is called a blob (which originally did not stand for anything but people have reverse engineered an acronym of Binary Large OBject). The storage system does not care about the contents of the blob.

You then store the blob with enough metadata (which you define) to enable you to understand what the blob is. The storage system gives it a unique ID which allows you to reference it.

To store an object, you create a bucket. The bucket helps define security and access controls. You then place your object in the bucket.

You might think of a bucket like a folder but it more like a volume. You are not expected to create lots of buckets or even to create them programatically. Buckets are flat and have no hierarchy.

## Access to the storage

File systems are accessed via your operating system. Depending on which operating system you are running, you will access your files in different ways. In fact, the features offered by the file system will differ for each operating system.

This means that when you are writing an application, you may have portability issues across operating systems and may have to build different versions as a result.

Object storage is accessed via a REST API call. Using POST, PUT, GET to an HTTP endpoint, you can create, read, update and delete (the famous CRUD operations) your blobs of data.

==Use of REST APIs means that, regardless of the operating system you are running on, your access to your object storage is the same.==

## Advantages

Object Storage provides a number of advantages over a filesystem:

- Storage of unstructured data in an unstructured way
- Unlimited storage
- ^ Extremely high availability
- ^ High performance
- Application technology independent access
- Versioning
- Object locking

^ These depend on the deployment architecture of your object storage.

## Object storage options

Ok, if object storage is so good, why isn’t everyone using it? Well, they pretty much are. AWS S3 Buckets are probably one of the best know object storage services with almost unlimited storage, an 11 nines uptime (that is 99.999999999% uptime or a fraction of a second downtime per year on average).

Just to re-enforce that buckets are not something to be created like confetti, AWS limit the number of buckets per account but that should not matter for most applications.

Oracle, Microsoft, Google and other cloud providers also have object storage solutions.

But what if you do not want to use a public storage solution? Your Kubernetes cluster may be on premise, your data may be sensitive or you may just want to develop against an object storage system without the cost of a 3rd party supplier.

This is where MinIO comes in.

# MinIO

MinIO is a full-service object storage solution that you can install yourself. The great thing about MinIO is that it has an API that is compatible with the AWS S3 interface. This means that you can swap from a MinIO solution to an AWS S3 solution (or vice versa).

MinIO can be installed in three ways:

- Single-node, single-drive (SNSD) or standalone
- Single-node, multi-drive (SNMD)
- Multi-node, multi-drive (MNMD)

These are quite self-explanatory. It should also be obvious that SNSD is probably the easiest to install but least resilient/available whilst MNMD is the hardest to install but most resilient/available.

For development and exploration, we will implement a standalone (SNSD) solution. For an enterprise solution you probably want to look at MNMD.

## Use with Kubernetes

To those of you paying attention, you probably saw the opening diagram that shows our MinIO outside the Kubernetes cluster. You may be thinking that you could implement a full MNMD solution within Kubernetes and you would be right.

You can install MinIO on your cluster using a Helm operator chart. If you are not familiar with operators, they are pods that manage the application state (as opposed to the container orchestration itself). The operator does most of the heavy lifting for you and you can create enterprise ready solutions quickly.

In any storage solution, you need to consider where your data is being stored. Data is your most valuable asset and, if you lose it, it could cost you dearly.

In my articles I am introducing you to technology and concepts but at the same time I want you to think about some of the design decisions you make.

I like the idea of having my storage solution in a place where I know I can back it up. This is similar to having stateless application servers backed by a database. You know where your data is, you know what you need to scale and you know what to protect. By deploying MinIO to your Kubernetes cluster, you will be distributing your data across that cluster.

If you have been following along with my articles, you will know that I am creating a Kubernetes cluster on a bare bones cloud provider ([Binary Lane](https://binarylane.com.au)) along with all the infrastructure required to manage and operate it. This looks like this:

![1_OkK__DUxFuSuNpDiXzt9Qg.webp]

From the architecture you can see that I already have a separate server providing my NFS share to the cluster, without being in the cluster itself. All my files are now in one place and available to the cluster, regardless of the number of nodes and pods I may have.

This makes the `nfs-server` the ideal place to install MiniIO, ensuring all my data is in this one place.

Also, remember that MinIO works best when it has direct access to the disk drives underpinning the storage solution rather than working via another share technology, such as NFS.

Of course, failure of that server means loss of all my data, however, I ensure that this server has a full back up taken by Binary Lane every day. If I needed more reliability and availability than this, I can always look to create a storage cluster if required.

So for this article, we will install a SNSD Standalone MinIO solution on my `nfs-server` and make it available to the cluster.

## Installing MinIO

As mentioned, we will be installing MinIO on the `nfs-server`. Log in to that server. These commands need to be **_run as root_**.

First we need a user to run MinIO. Create the user as follows:

```sh
groupadd -r minio-user  
useradd -M -r -g minio-user minio-user
```

This creates a group and a user called `minio-user`. The user will be a system user (`-r`) has no home folder (`-M`).

Now we create the folder where we want MinIO to store its data and we will make the new user the owner of that folder.

```
mkdir /mnt/minio  
chown minio-user:minio-user /mnt/minio
```

The next step is to download the deb package and install it:

```
wget https://dl.min.io/server/minio/release/linux-amd64/archive/minio_20240111074616.0.0_amd64.deb -O minio.deb  
dpkg -i minio.deb
```

Check it installed with:

`minio -v`

We now need to set up the configuration file. Create this file, replacing fields between < and > with your own values:

`**/etc/default/minio**`

```sh
# define where the data is to be held  
MINIO_VOLUMES="/mnt/minio"  
  
# define where to find the TLS certificates  
#MINIO_OPTS="--certs-dir /etc/minio/certs --console-address :9001"  
  
# define the admin user (min 3 characters)  
MINIO_ROOT_USER=<minio admin user>  
  
# define the defaut admin user password (min 8 characters)  
MINIO_ROOT_PASSWORD=<minio admin password>
```

Note that the options line is commented out as we will not be implementing TLS connections at this stage.

There is a lower limit of 3 characters on the username and 8 for the password.

You may want to allow ports 9000 (user interface) and 9001 (API) through your firewall if you have one. In my case I do not.

With the configuration all in place, we will start MinIO as a service. You can run it in the foreground but that is not advised as it can cause problems when you come to run it as a service.

The installation already set up a service configuration so you can start it, check its status and enable it.

```sh
systemctl start minio  
systemctl status minio  
systemctl enable minio
```

With the system started, you can now navigate to `http:<nfs-server IP address>:9000`

## Testing the deployment

When you navigate to the address above, you should be presented with a login screen. Note that MinIO uses the same port for browser and API access. Use the credentials you added to the configuration file earlier.

Whilst you can explore the large range of options available via the menu, there are two important ones:

1. `Buckets`: allows you to create and manage buckets within your MinIO deployment
2. `Access Keys`: allows you to create secure access keys for use with your application

First create a new bucket. On the MinIO console:

- Select `Buckets` from the left hand panel
- Click `Create Bucket +`
- Enter a name, eg: `Test`
- Click `Create Bucket`

Congratulations, you have just created your first bucket.

Now, to test the service you will need an Access Key. This is necessary to be able to access you MinIO REST API.

- Select Access Keys from the left hand panel
- Click on `Create access key +`.
- Name your new access key, eg: `Test.`
- Click on `Create`
- Securely note down the access key and secret key

Here is an example key and secret (don’t worry these have been made up):

```
Access: 1ndCLbxwcPLr9iHmwGYc
Secret: vgnnm5JEYV3sszGiFz2syefEtN2JUIVPKP1MtNih
```

Postman is my tool of choice for testing any API. In the following, remember to replace < > fields with your own values.

- Add a new `GET` request
- Enter the address: `http:<nfs-server IP address>:9000/<bucket name>`
- Under Authentication, select `AWS Signature`
- Enter your `AccessKey`
- Enter your `SecretKey`
- Enter a Service Name of `s3`
- Click `Send`

You should get a response with some details of your bucket, I get this:

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<ListBucketResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/">  
    <Name>test</Name>  
    <Prefix></Prefix>  
    <Marker></Marker>  
    <MaxKeys>1000</MaxKeys>  
    <IsTruncated>false</IsTruncated>  
</ListBucketResult>
```

Let’s put something in our bucket. Create this file on your development machine:

`**hello-world.txt**`

```
Hello World
```

Now let’s upload this through the console.

- Go to `Buckets`
- Click on your bucket (eg: `Test`)
- In the top right, click on the browse icon (looks like a folder)
- Click `Upload` -> `Upload File`
- Select your `hello-world.txt` file

Your file will be uploaded as an object. Now you can access it in your Postman with a `GET` request to `/<bucket name>/hello-world.txt`.

You should see `Hello World` come back.

Congratulations, you have now successfully installed MinIO.

## Securing the connection

You may have noticed that we are accessing our MinIO server via HTTP without any TLS encryption. **_This is not advised_** as anyone with access to the network will be able to see your data.

To secure our connections, we will install a self-signed certificate into the MinIO configuration. I am assuming you have a Certificate Authority set up. If not, you can read my article on [creating your own Certificate Authority (CA)](https://medium.com/@martin.hodges/introduction-to-creating-a-ca-on-debian-11-094bde1c676a).

First create a Certificate Signing Request (CSR) using `openssl`:

`openssl req -newkey rsa:2048 -keyout private.key -out minio.csr -nodes`

Note the `-nodes` option, which refers to no DES, ie: do not encrypt the private key and lock it with a passphrase.

You will now have your private key (`private.key`) and your Certificate Signing Request (`minio.csr`). Now sign your CSR by your CA and bring back the certificate.

You should have two files now:

```
minio.crt  
private.key
```

You will also need your CA certificate (`ca.crt`).

Now modify your MinIO configuration file by removing the comment from the options line. Replace server-admin with a non-root user on your system.:

**/etc/default/minio**

```
# define where the data is to be held  
MINIO_VOLUMES="/mnt/minio"  
  
# define where to find the TLS certificates  
MINIO_OPTS="--certs-dir /etc/minio/certs --console-address :9001"  
  
# define the admin user (min 3 characters)  
MINIO_ROOT_USER=<minio admin user>  
  
# define the defaut admin user password (min 8 characters)  
MINIO_ROOT_PASSWORD=<minio admin password>
```

Now create the path and copy your certificate files over and change to the ownership.

```sh
sudo mkdir -p /etc/minio/certs  
sudo cp minio.crt /etc/minio/certs/public.crt  
sudo cp private.key /etc/minio/certs  
sudo chown -R minio-user:minio-user /etc/minio/certs
```

With the certificates and keys loaded and the configuration updated we can now restart the MinIO server:

```sh
sudo systemctl restart minio  
sudo systemctl status minio
```

If you were paying attention, you will also have seen the addition of the `-- console-address :9001` configuration. This separates the UI and API ports, which is a good practice.

Now, when you go to your console (at port `9001`), you will see that the site is described as ‘Not Secure’ and you will need to convince your browser to go to the site. This is because you are using private certificates. You will need to add the `ca.crt` certificate to the browser store. I will leave you to work out how to do that.

Depending on your Postman settings, a private certificate may or may not be accepted and you may need to add your ca.crt certificate to Postman too.