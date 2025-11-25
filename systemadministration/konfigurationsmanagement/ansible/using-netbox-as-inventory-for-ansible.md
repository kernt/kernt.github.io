---
tags:
  - konfigurationsmanagement
  - ansible
  - netbox
  - inventory
---

There are a lot of tutorial which show how to synchronize inventory items or items `facts`in Netbox, today I propose the opposite exercise: using Netbox as an Ansible inventory 
source.

Before going into detail, I thought that a little reminder of the main protagonists of this adventure would not hurt to refresh our ideas.
# Ansible

Ansible is a deployment tool that requires few dependencies on the target machines to be deployed. They are often limited to SSH and Python, it is often called _agentless_.

The deployment recipe is described as follows:

- On the one hand, in YAML, with the step-by-step code (tasks) ensuring the installation of packages and the deployment of files on the target machines;
- On the other hand, with the inventory describing the specificities linked to the desired state of each target machine (variables).

For more complex deployments, inventory structure can become a real problem. Here’s an example:

`host.ini`

```
server1 ansible_host=10.0.1.2   
server2 ansible_host=10.0.2.1
db   ansible_host=10.0.1.2
```

Although perfectly valid from Ansible’s point of view, `server1`and `db`have the same IP address. This is an easy error to spot in a 3-line inventory, but it becomes more difficult when the inventory (or inventories!) is several hundred lines long.

Although perfectly valid from Ansible’s point of view, `server1`and `db`have the same IP address. This is an easy error to spot in a 3-line inventory, but it becomes more difficult when the inventory (or inventories!) is several hundred lines long.
# Netbox

Netbox is a software that has made inventory its core business . It ensures data integrity while providing a clear interface for viewing and editing. By using it, inventory errors like the ones above can be easily identified and avoided.

The other interesting aspect of netbox is its API allowing the exploitation of this rich source of data by other software.

If you want to get your hands dirty, you can follow [this tutorial using Docker Compose](https://docs.docker.com/compose/install/) and the Git repository below:

```sh
$ git clone -b release https://github.com/netbox-community/netbox-docker.git                                                       
$ cd netbox-docker  
$ tee docker-compose.override.yml <<EOF  
version: '3.4'  
services:  
  netbox:  
    ports:  
      - 8000:8080  
EOF                                                    
$ docker-compose up -d
$ export NETBOX_API="http://127.0.0.1:8000" # NETBOX_API sera l'url de netbox dans la suite du tuto
```

# Netbox, in inventory for Ansible

When working with configuration information stored in duplicate in multiple different systems, it is generally relevant to decree that only one of these systems will be the _Single Source Of Truth_. In other words, it will be the one that will be authoritative, and the others will cascade or synchronize on it. Here we propose that Netbox be authoritative, and that Ansible synchronizes on it thanks to the Netbox API.

# Token between Netbox and Ansible

For Netbox, this is about recognizing Ansible as a valid user capable of reading the data. If Ansible were a human capable of using a browser, we would give it a username and password. Since Ansible is not a human, we will instead use a token. If you installed Netbox with the repository above, a token was automatically created along the way with a hard-coded value, and we can use it as is for the rest:

```sh
export NETBOX_TOKEN=0123456789abcdef0123456789abcdef01234567                       
curl -H "Authorization: Token $NETBOX_TOKEN" "${NETBOX_API}/api/dcim/sites/"
```

Otherwise, to test on an existing Netbox (in which, hopefully, this token does not exist yet), you need to log in to this Netbox (button at the top right) with an administrator account. Then you need to go to the “admin” part of Netbox (via the same menu at the top right, which should now show an “Admin” entry, precisely). The “USERS” section of the back-office includes a “Token” entry, which allows you to list existing tokens; as well as to add a token (“Add token”). A simple token with read-only permissions is required for this Ansible-based tutorial.

Once the token is obtained, we can test it with the following command:

```sh
export NETBOX_TOKEN=XXXXXXXXXXXXXXXXXXXXXXXXXXXXX
curl -H "Authorization: Token $NETBOX_TOKEN"
"${NETBOX_API}/api/dcim/sites/"
```

> If your Netbox installation is in HTTPS but you get URLs starting with “http://<host>” in your responses, you probably have a common reverse proxy problem.

It should still work, but you will make twice as many requests (the first being in HTTP, then redirected to HTTPS). Worse: you send your token in clear text, one request out of 2! You can do a simple _curl https://netbox.example.com/api/_ , to check.

# Creating Netbox Inventory

_netbox.yml_

```yaml
---
plugin: netbox.netbox.nb_inventory 
query_filters: 
  - status: active
```

Then install the Netbox collection (requiring Ansible > 2.9):

`ansible-galaxy collection install netbox.netbox`

At this point, if you are testing with a Netbox that already contains your servers or other equipment, you should see them appear in your Ansible inventory.

On the other hand, with a freshly installed Netbox (as from the git repository given as an example above), the Ansible inventory is empty because there is nothing declared in this Netbox yet. If necessary, we can fill our Netbox a little to have some information:

- Create one `site`(with a name)
- Create one `device-role`(with a name)
- Create one `manufacturer`(with a name)
- Create a `device-type`(with a name and a `manufacturer`)
- Create a `device`(with a name, a `device-role`, a `device-type`and a site)
- Click on your device, top right `Add Components`> `Interface`, add one `interface`(with a name)
- Click on your interface, add an IP address (with an IP address followed by a slash “/” and a netmask! For example: “127.0.0.1/8”)

Pfieww! Yes, Netbox requires a minimum of classification and does not let us create elements without a minimum of information, but it is this requirement that makes all the power of this software.

Let’s check if we now find the fruit of our hard click work in Netbox via the _ansible-inventory_ command.

```
$ ansible-inventory -i netbox.yml --list
{                                                                                          
    "_meta": {                                                                             
        "hostvars": {                                                                  
            "laptop": {                                                                  
                "ansible_host": "127.0.0.1"
                "custom_fields": {},                          
                "device_roles": [ 
                    "test"                                 
                ],  
                "device_types": [
                    "xps-13"                                                        
                ],                                                                                                
                "is_virtual": false,                                            
                "local_context_data": [                                                                        
                    null                                                                                                              
                ],                                                                                                       
                "manufacturers": [                                                                                                                                                                    
                    "dell"                                                                                                                                                                            
                ],
                "primary_ip4": "127.0.0.1",
                "regions": [],
                "services": [],
                "sites": [       
                    "local" 
                ],                                                              
                "tags": []                          
            } 
        }
    },
    "all": {                                    
        "children": [
            "ungrouped"     
        ]
    }, 
    "ungrouped": {
        "hosts": [                                                                
            "laptop"
        ]                                                                             
    }
}
```

https://medium.com/@talhakhalid101/using-netbox-as-inventory-for-ansible-d172c6925d96

Ahhhh, people, finally!

# A little order in this group world

On the other hand, all our equipment is in a group called “ungrouped”, itself in the “all” group, in other words… there is no group.

Not a big deal for a demo where you only have one piece of equipment, but if you have a well-populated Netbox, you’ll probably want to organize things a little better.

Let’s solve this problem with the option `group_by`:

`netbox.yaml`

---                                                                                                                                                                                                    
plugin: netbox.netbox.nb_inventory                                                                                                                                                                    
query_filters:                                                                                                                                                                                        
  - status: active                                                                                                                                                                                    
group_by:                                                                                                                                                                                              
  - device_roles                                                                                                                                                                                      
  - sites

The list of groups becomes:

$ ansible-inventory -i netbox.yml --graph                                                                                                                                                                
@all:                                                                                                                                                                                                  
  |--@sites_local:                                                                                                                                                                                    
  |  |--laptop                                                                                                                                                                                        
  |--@device_roles_test:                                                                                                                                                                              
  |  |--laptop                                                                                                                                                                                        
  |--@ungrouped:

Aren’t we good here? No, not quite; the group `sites_local`is still going, but am I going to have to deal `device_roles_`with each group of devices_roles? Not necessarily:

`netbox.yml`

---  
plugin: netbox.netbox.nb_inventory                                                                                                                                                                    
query_filters:                                                                                                                                                                                        
  - status: active                                                                                                                                                                                    
group_by:                                                                                                                                                                                              
  - sites                                                                                                                                                                                              
keyed_groups:                                                                                                                                                                                          
  - prefix: ""                                                                                                                                                                                        
    separator: ""                                                                                                                                                                                      
    key: device_roles[0]

The setup may seem weird, but it is perfectly functional.

$ ansible-inventory -i netbox.yml --graph  
@all:   
  |--@site_local:  
  |  |--laptop  
  |--@test:  
  |  |--laptop  
  |--@ungrouped:

# Storing the token with Ansible-vault

The plugin `netbox.netbox.nb_inventory`relies on the two environment variables we defined above, `NETBOX_URL`and `NETBOX_TOKEN`.

If we use a system like Kubernetes, or a CI/CD platform, there are usually relatively simple ways to define environment variables; but in the particular case of Ansible, how can we store these variables (in particular the token) securely?

Ansible has a sensitive information storage system: “ansible-vault”. We will not go into the details of the [ansible-vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) setup, it is of little interest for this blog post, but to give you a little idea, you can encrypt your token like this:

$ ansible-vault encrypt_string "${NETBOX_TOKEN}" --name 'token''  
token: !vault |  
  $ANSIBLE_VAULT;1.1;AES256  
  XXXXXXXXXXXXXXXXXXX  
  XXXXXXXXXXXXXXXXXXX

Then integrate it into the inventory file:

`netbox.yml`

---  
plugin: netbox.netbox.nb_inventory  
api_endpoint: https://demo.netbox.dev/  
token: !vault |  
  $ANSIBLE_VAULT;1.1;AES256  
  XXXXXXXXXXXXXXXXXXX  
  XXXXXXXXXXXXXXXXXXX                                                                                                                                                               query_filters:                                                                                                                                                                                        
  - status: active                                                                                                                                                                                    
group_by:                                                                                                                                                                                              
  - sites                                                                                                                                                                                              
keyed_groups:                                                                                                                                                                                          
  - prefix: ""                                                                                                                                                                                        
    separator: ""                                                                                                                                                                                      
    key: device_roles[0]

# To go further

The inventory allows you to have generic information (name, site, main IP address, etc.) of a device. On the other hand, if you want to get all the information of a device (for example, have access to all the interfaces of a server and not only the main one), you will have to make additional Netbox requests. You can use the _nb_lookup_ lookup (delivered with the netbox.netbox collection) in roles when you need to work with it.

Finally, in hybrid infrastructures where Cloud is mixed with “On-premises”, we can ensure that information from the Cloud is replicated in Netbox with Terraform and [its provider](https://github.com/smutel/terraform-provider-netbox) (there are others but this is the one that has given us the most satisfaction at Enix).

There you go, you normally have all the tools to set up Ansible with a Netbox inventory! And potentially then the other systems too?