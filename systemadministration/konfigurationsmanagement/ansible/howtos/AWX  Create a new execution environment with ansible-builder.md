In my previous post, I deployed ansible awx via awx-operator into a k3s cluster. After several deployments, I have had to use the `execute_lambda` from community.aws collection. I don’t know why but I was able to use different functions from this collection but `execute_lambda`was impossible to call.

So finally I decide to find a solution and I would like to share my experience with you :-)

## List your collection in awx-operator

As I can read, it is not recommended to add a collection in AWX-EE default.

You can list your collection in your current cluster with the following command :

```sh
kubectl get pods -n awx  
kubectl exec -it awx-64bc58f8d6-74pvx -n awx -c awx-ee -- /bin/bash -c "ansible-galaxy collection list"  
  
# /usr/share/ansible/collections/ansible_collections  
Collection              Version  
----------------------- -------  
amazon.aws              3.0.0  
ansible.posix           1.3.0  
ansible.windows         1.8.0  
awx.awx                 19.4.0  
azure.azcollection      1.10.0  
community.vmware        1.17.0  
google.cloud            1.0.2  
kubernetes.core         2.2.2  
openstack.cloud         1.5.3  
ovirt.ovirt             1.6.6  
redhatinsights.insights 1.0.5  
theforeman.foreman      3.0.0
```

If the collection you need is not in the list, you have to create your custom awx-ee image.

And that’s why Ansible-builder exists.

## Set up ansible-builder

First create a virtualenv python3.

For ubuntu 20.04, you have to set up python3, pip, virtualenv

```sh
sudo apt update  
sudo apt install python3-pip  
sudo apt install virtualenv  
virtualenv .venvpy3  
pip install ansible-builder
```

Now you have a virtualenv python3 and ansible-builder installed.

## Create your custom awx-ee image

In order to create an image you have to create few files.

Ansible-builder can take different type of requirements to install in your custom awx-ee :

- requirements.yml : Ansible-galaxy collection to install
- requirements.txt : python dependencies to install
- bindep.txt: system dependencies to install

In my case I only want to add community.aws and community.general to my custom image.

So I create a requirements.yml :

```json
---  
collections:  
- awx.awx  
- amazon.aws  
- ansible.posix  
- ansible.windows  
- awx.awx  
- azure.azcollection  
- community.vmware  
- google.cloud  
- kubernetes.core  
- openstack.cloud  
- ovirt.ovirt  
- redhatinsights.insights  
- theforeman.foreman  
- community.aws  
- community.general
```

Some of collections are by default in awx-ee, so I retrieve them to be sure my custom image works exactly as awx-ee does.

You have to create a file execution-environment.yml. It is the file execute by ansible-builder by default.

```json
---  
version: 1  
dependencies:  
  galaxy: requirements.yml
```

And you can finally launch ansible-builder :

`ansible-builder build --tag=custom-awx-ee --container-runtime=docker --verbosity=3`

By default it is pod man that is used as container-runtime but you can define docker with the option above.

At the end, you can find your custom image with a `docker images`command.

To be able to use your custom image, you have to push it to your container registry.

In my case I use gitlab private registry.

## Use your custom awx-ee in AWX

Once your custom image is pushed to your registry, you have to :

- configure a new execution environment in awx.
- define a new project with your custom execution environment
- define a template with your custom execution environment

And you ready to launch your first job using your custom awx-ee.

If you have any question and I can hel, ask with a comment.