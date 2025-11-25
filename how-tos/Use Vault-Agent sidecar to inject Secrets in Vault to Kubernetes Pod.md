---
tags:
  - vault
  - sicherheit
  - kubernetes
---
In our previous post, we managed to [install Vault cluster in GKE](https://computingforgeeks.com/install-vault-cluster-gke-via-helm-terraform-bitbucket-pipelines/) and we were not able to add Kubernetes Authentication to it.
This post takes you into a different vehicle that will take us through the adding Kubernetes Auth and also add a sample application that we intend to get credentials from Vault. We shall proceed step by step and by the end of the guide, we should have the secrets injected into our Pod in Kubernetes (GKE).
## Project Pre-requisites

- A working Kubernetes Cluster
- Vault already installed, unsealed and running.
## Add Kubernetes Authentication Method to Vault

The token is the one we had after unsealing the Vault leader in the cluster in the previous post.

**We shall first exec into the vault-0 container:**

```sh
kubectl exec -it vault-0 -n vault /bin/sh
```

**Then in the shell prompt that ensues, login to Vault and enable Kubernetes Auth. Use the root token to login to vault.**

```sh
vault login

Token (will be hidden): <Enter your token>

Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                s.scramblskdfhwkeuigpb
token_accessor       lskdfhwm3zh9blskdfhw0E
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```

**List raft peers just to confirm everything is okay.**

```sh
/ $ vault operator raft list-peers

Node                                    Address                        State       Voter
----                                    -------                        -----       -----
71269676-b59e-301a-c054-86a844228bca    vault-0.vault-internal:8201    leader      true
6e0d5183-3896-a73b-be7c-300a61b8a218    vault-1.vault-internal:8201    follower    true
ee238252-5aa5-e5ed-730d-ccb8d6334b1d    vault-2.vault-internal:8201    follower    true
d24854db-05f8-a3ec-f32a-d4e5897ea8db    vault-3.vault-internal:8201    follower    true
c0c774c8-5737-a3c7-cbd7-a838249fafee    vault-4.vault-internal:8201    follower    true
```

Now enable Kubernetes Auth method

```sh
vault auth enable kubernetes

Success! Enabled kubernetes auth method at: kubernetes/
```
## Configure Kubernetes Auth Method

In this step, we are going to bootstrap Kubernetes auth method we enabled in **_Step 1_** by adding relevant configuration details. We will be running commands inside a Vault pod running in Kubernetes.

**You will optionally need the following variables:**

```sh
# JWT is a service account token that has access to the Kubernetes TokenReview API
# You can retrieve this from inside a pod at: /var/run/secrets/kubernetes.io/serviceaccount/token
JWT=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)

# Address of Kubernetes itself as viewed from inside a running pod
KUBERNETES_HOST=https://${KUBERNETES_PORT_443_TCP_ADDR}:443

# Kubernetes internal CA
KUBERNETES_CA_CERT=$(cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt)
```

**Exec/Login into the Vault pod:**

```sh
kubectl exec -it vault-0 /bin/sh
```

**Then run the following command to configure the Kubernetes Auth Method:**

```sh
/ $ vault write auth/kubernetes/config \
    token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
    kubernetes_host=https://${KUBERNETES_PORT_443_TCP_ADDR}:443 \
    kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt disable_iss_validation=true \
    issuer="https://kubernetes.default.svc.cluster.local"

Success! Data written to: auth/kubernetes/config
```

The “_token_reviewer_jwt_” and “_kubernetes_ca_cert”_ are mounted to the container by Kubernetes when it is created. The environment variable _KUBERNETES_PORT_443_TCP_ADDR_ is defined and references the internal network address of the Kubernetes host.
## Add secrets, policy and role to Vault

In this Step, we are going to add a secret to a path in Vault which we shall enable soon. We shall then create a policy that grants specific capabilities to the path such as read, write and others. Some sort of permissions governing how the secret will be accessed. We shall thereafter create a role that will use the policy having the requisite permissions. Let us do this:

**First, we shall enable or create a path (secret in this example) that will hold our secrets**

```sh
/ $ vault secrets enable -path=secret kv-v2

Success! Enabled the kv-v2 secrets engine at: secret/
```

**After that, let us write a sample username and password secret to the path**

```sh
/ $ vault kv put secret/devwebapp/config username='giraffe' password='salsa'

Key              Value
---              -----
created_time     2021-11-11T14:38:09.892584386Z
deletion_time    n/a
destroyed        false
version          1
```

**Make sure that the username and password was written accordingly by listing**

```sh
/ $ vault kv get secret/devwebapp/config

====== Metadata ======
Key              Value
---              -----
created_time     2021-11-11T14:38:09.892584386Z
deletion_time    n/a
destroyed        false
version          1

====== Data ======
Key         Value
---         -----
password    salsa
username    giraffe
```

Beautiful, our username and password was updated successfully.

Next, you remember the policy with permissions we mentioned? We are going to create it now. We will call the policy “_demoapp_” And as you can see, it references the path we created earlier.

```sh
/ $ vault policy write demoapp - <<EOF
  path "secret/data/devwebapp/config" {
    capabilities = ["read"]
 }
EOF

Success! Uploaded policy: demoapp
```

Then, the policy will be useless if there are no clients to use it. Roles are the objects that policies are applied to. Another thing about roles is that they are attached to specific Kubernetes service account that ensures that only specific service accounts in Kubernetes cluster will be able to receive the secret stored in Vault. So let us create a role, apply the policy to it and assign it to a service account. Do not worry if the service account is not yet provisioned in your Kubernetes Cluster, we will create it later.

Briefly, the role “_devweb-app_” connects a Kubernetes service account, “_internal-app_“, and namespace “_vault_“, with the Vault policy, “_demoapp_“.

Here we go:

```sh
/ $ vault write auth/kubernetes/role/devweb-app \
        bound_service_account_names=internal-app \
        bound_service_account_namespaces=vault \
        policies=demoapp \
        ttl=24h
Success! Data written to: auth/kubernetes/role/devweb-app
```

We have set the time for the token after authentication to expire in 24 hours. You will also notice that we have specified the namespace that the service account will reside in.

You can now exit the vault container terminal 
## Create Service Account and Sample application.

Create a Kubernetes service account named internal-app if it does not exist already in vault namespace.

```sh
kubectl create sa internal-app -n vault
```

Create your deployment file with vault’s details added. An example is shown below that will deploy Nginx and add inject the secret in the Vault cluster right into the pod via a sidecar.

As you can see, we have added “_vault.hashicorp.com_” annotations with the details we had set before in Vault such as the name of the role “_devweb-app_“, the serviceAccount, the secret path and that we want a side-car injected.

```sh
vim nginx-sample-deployemnt.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-vault-demo
  namespace: vault
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      annotations:
        vault.hashicorp.com/agent-inject: 'true'
        vault.hashicorp.com/role: 'devweb-app'
        vault.hashicorp.com/agent-pre-populate: "false"
        vault.hashicorp.com/agent-inject-secret-config.txt: 'secret/data/devwebapp/config'
      labels:
        app: nginx
    spec:
      serviceAccountName: internal-app
      containers:
        - name: nginx
          image: nginx:latest
```

**Apply the manifest to Kubernetes**

```sh
kubectl apply -f nginx-sample-deployemnt.yaml
```

**Check if the vault-agent pods injected the credentials to your pod**

```sh
kubectl logs nginx-vault-demo-86c599c6cd-8hch9 -n vault -c vault-agent

==> Vault agent started! Log data will stream in below:

==> Vault agent configuration:

                     Cgo: disabled
               Log Level: info
                 Version: Vault v1.8.4
             Version Sha: 925bc650ad1d997e84fbb832f302a6bfe0105bbb

2021-11-11T20:20:50.506Z [INFO]  sink.file: creating file sink
2021-11-11T20:20:50.506Z [INFO]  sink.file: file sink configured: path=/home/vault/.vault-token mode=-rw-r-----
2021-11-11T20:20:50.507Z [INFO]  template.server: starting template server
2021-11-11T20:20:50.507Z [INFO] (runner) creating new runner (dry: false, once: false)
2021-11-11T20:20:50.507Z [INFO] (runner) creating watcher
2021-11-11T20:20:50.507Z [INFO]  auth.handler: starting auth handler
2021-11-11T20:20:50.508Z [INFO]  auth.handler: authenticating
2021-11-11T20:20:50.508Z [INFO]  sink.server: starting sink server
2021-11-11T20:20:50.560Z [INFO]  auth.handler: authentication successful, sending token to sinks
2021-11-11T20:20:50.561Z [INFO]  auth.handler: starting renewal process
2021-11-11T20:20:50.561Z [INFO]  sink.file: token written: path=/home/vault/.vault-token
2021-11-11T20:20:50.561Z [INFO]  template.server: template server received new token
2021-11-11T20:20:50.561Z [INFO] (runner) stopping
2021-11-11T20:20:50.561Z [INFO] (runner) creating new runner (dry: false, once: false)
2021-11-11T20:20:50.568Z [INFO] (runner) creating watcher
2021-11-11T20:20:50.568Z [INFO] (runner) starting
2021-11-11T20:20:50.579Z [INFO]  auth.handler: renewed auth token
2021-11-11T20:20:50.684Z [INFO] (runner) rendered "(dynamic)" => "/vault/secrets/config.txt"
```

As you can see, the runner “rendered” the config file dynamically to the pod

**Let us login to the container and check the configuration file under “/vault/secrets/config.txt”**

```sh
$ kubectl exec -it nginx-vault-demo-86c599c6cd-8hch9 -n vault /bin/sh

# ls /vault/secrets
config.txt
```

**And there we have our credentials file. We can “cat” it and check if it has the credentials we actually configured in the Vault cluster**

```sh
# cat /vault/secrets/config.txt
data: map[password:salsa username:giraffe]
metadata: map[created_time:2021-11-11T14:38:09.892584386Z deletion_time: destroyed:false version:1]
```
## Create a template within the secrets file

As you can see from the previous Step, Vault writes the secret in a manner that can be hard for applications to read. Luckily, there is an annotation we can add that will format it in a way that we want it. We will simply add one more annotation as follows:

```sh
vim nginx-sample-deployemnt.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-vault-demo
  namespace: vault
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      annotations:
        vault.hashicorp.com/agent-inject: 'true'
        vault.hashicorp.com/role: 'devweb-app'
        vault.hashicorp.com/agent-pre-populate: "false"
        vault.hashicorp.com/agent-inject-secret-config.txt: 'secret/data/devwebapp/config'
        vault.hashicorp.com/agent-inject-template-config.txt: |
          {{- with secret "secret/data/devwebapp/config" -}}
          USERNAME={{ .Data.data.username }}
          PASSWORD={{ .Data.data.password }}
          {{- end -}}
      labels:
        app: nginx
    spec:
      serviceAccountName: internal-app
      containers:
        - name: nginx
          image: nginx:latest
```

**Now that the edits are done, we can proceed to apply this new manifest to Kubernetes as usual**

```sh
kubectl apply -f nginx-sample-deployemnt.yaml
deployment.apps/nginx-vault-demo configured
```

**View new pod**

```sh
kubectl get pods -n vault 

NAME                                    READY   STATUS        RESTARTS   AGE
nginx-vault-demo-64fb65bb4-g5n9n        2/2     Running       0          8s
vault-0                                 1/1     Running       0          6d19h
vault-1                                 1/1     Running       0          6d19h
vault-2                                 1/1     Running       0          6d19h
vault-3                                 1/1     Running       0          6d19h
vault-4                                 1/1     Running       0          6d19h
vault-agent-injector-7b59447b69-bh9sv   1/1     Running       0          4d15h
```

**You can now exec into the new pod and check if the file was updated accordingly.**

```sh
kubectl exec -it nginx-vault-demo-64fb65bb4-g5n9n -n vault /bin/bash
```

```sh
root@nginx-vault-demo-64fb65bb4-g5n9n:/# more /vault/secrets/config.txt

USERNAME=giraffe
PASSWORD=salsa
```

Beautiful. Now you can extract those fields that are formatted properly and use them in your application.

**_Books For Learning Kubernetes Administration:_**

- [Best Kubernetes Study books](https://computingforgeeks.com/best-kubernetes-study-books/
## End Note

We have managed to use the Vault Cluster we built earlier by using a real application that received secrets saved far away in Vault. Now you can enjoy the separation of concerns and know that your secrets are well kept in Vault. There is no need anymore to upload your env files to git. Just place all of the secrets in Vault and you can use the Sidecar model or you can code the logic directly so that your application can retrieve the secrets from Vault directly.

We thank your readership, your encouraging comments and the enormous support pouring in. Have a joyous end of the year season guys!!

Other guides you will enjoy:

- [Ansible Vault Cheat Sheet / Reference guide](https://computingforgeeks.com/ansible-vault-cheat-sheet-reference-guide/)
- [Install and Configure Hashicorp Vault Server on Ubuntu / CentOS / Debian](https://computingforgeeks.com/install-and-configure-vault-server-linux/)
- [How To Store Terraform State in Consul KV Store](https://computingforgeeks.com/how-to-store-terraform-state-in-consul-kv-store/)
- [How To Setup Consul Cluster on CentOS / RHEL](https://computingforgeeks.com/how-to-setup-consul-cluster-on-centos-rhel/)
- [Setup Consul Cluster on Ubuntu & Debian](https://computingforgeeks.com/how-to-install-consul-cluster-18-04-lts/)