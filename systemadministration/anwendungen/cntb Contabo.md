---
tags:
  - contabo
  - cntb
---
# cntb Allgemein

* [cntb](https://github.com/contabo/cntb)

https://github.com/contabo/cntb/releases/download/v1.4.12/checksums.txt

```sh
Available Commands:
  assign                  Add instance to private network
  cancel                  Cancel an existing subscription
  completion              Generate completion script
  config                  Manage config files
  create                  Create a new resource
  delete                  Deletes a resource
  edit                    Edits an existing resource
  generateSecret          Generate a secret for resources
  get                     Show one or more resources
  help                    Help about any command
  history                 Shows history of your resources
  regenerate              regenerate specific resource
  reinstall               Reinstall an existing instance
  rescue                  rescue an existing instance
  resendEmailVerification Resend email verification
  resetPassword           Send password reset email
  resize                  Resize an existing object storage.
  restart                 Restarts a resource
  rollback                Rollback a resource
  shutdown                Shutdown a resource
  start                   Start a resource
  stats                   get stats of an resource
  stop                    Stop a resource
  unassign                Remove instance from private network
  update                  Updates an existing resource
  upgrade                 upgrade an existing instance.
  version                 Shows the version and exits
```

# cntb einrichten

Configure `cntb` once to use your credentials. You can obtain them from [Customer Control Panel](https://my.contabo.com/api/details).

```sh
cntb config set-credentials --oauth2-clientid=<ClientId from Customer Control Panel> --oauth2-client-secret=<ClientSecret from Customer Control Panel> --oauth2-user=<API User from Customer Control Panel> --oauth2-password=<API Password from Customer Control Panel>
```

**Enable Shell Completion**

```sh
cntb completion
Bash:
        $ source <(cntb completion bash)
        # To load completions for each session, execute once:
        # Linux:
        $ cntb completion bash > /etc/bash_completion.d/cntb
        # macOS:
        $ cntb completion bash > /usr/local/etc/bash_completion.d/cntb
Zsh:
        # If shell completion is not already enabled in your environment,
        # you will need to enable it.  You can execute the following once:
        $ echo "autoload -U compinit; compinit" >> ~/.zshrc
        # To load completions for each session, execute once:
        $ cntb completion zsh > "${fpath[1]}/_cntb"
        # You will need to start a new shell for this setup to take effect.
fish:
        $ cntb completion fish | source
        # To load completions for each session, execute once:
        $ cntb completion fish > ~/.config/fish/completions/cntb.fish
PowerShell:
        PS> cntb completion powershell | Out-String | Invoke-Expression
        # To load completions for every new session, run:
        PS> cntb completion powershell > cntb.ps1
        # and source this file from your PowerShell profile.
```
## cntb create

Available Commands:
  image          Creates a new image.
  instance       Create a new compute instance.
  role           Creates a new role.
  secret         Creates a new secret.
  snapshot       Creates a new snapshot
  tag            Creates a new tag
  tagAssignment  creates a new tag assignment
  user           Creates a new user

**User anlegen**

`cntb create user `

**Secret anlegen**

`cntb create secret --name 'ssh Privatekey' --value 'secret' --type 'ssh'`

**Upload custom image**

```cntb create image --name 'CentOS Cloud Image' --description 'CentOS 7 Cloud Image' --url 'https://cloud.centos.org/altarch/7/images/CentOS-7-x86_64-GenericCloud.qcow2' --osType Linux --version 7 ```

**Create / order new Compute Instance**

Using Cloud-Init to set ssh public key

```sh
cntb create instance --imageId "ae423751-50fa-4bf6-9978-015673bf51c4" --productId "V45" --region "EU" --userData 'ssh_authorized_keys:
  - ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAGEA3FSyQwBI6Z+nCSjUUk8EEAnnkhXlukKoUPND/RRClWz2s5TCzIkd3Ou5+Cyz71X0XmazM3l5WgeErvtIwQMyT1KjNoMhoJMrJnWqQPOt5Q8zWd9qG7PBl9+eiH5qV7NZ'
# once finished please login via ssh
# in case of the previously uploaded CentOS 7 Cloud Image please use `centos` as user
# for standard images please use `admin` as user
```

Using Cloud-Init to install apache2 with an already stored SSH public key

```sh
cntb create instance --imageId "ae423751-50fa-4bf6-9978-015673bf51c4" --productId "V45" --region "EU" --sshKeys '1,2' --userData 'package_update: true
package_upgrade: true
packages:
  - httpd'
```

**Insdancen ausgeben**

```sh
(venv) tobkern@srv3:~/git/k3s-ansible$ cntb get instances
  INSTANCEID  NAME       DISPLAYNAME  STATUS   IMAGEID                               REGION  PRODUCTID  IPV4           IPV6                                     DEFAULTUSER  
  100154798   vmd154798               running  4efbc0ba-2313-4fe1-842a-516f8652e729  EU      V45        86.48.5.122    2a02:c206:3015:4798:0000:0000:0000:0001  admin        
  100134720   vmd134720               running  4efbc0ba-2313-4fe1-842a-516f8652e729  EU      V45        144.91.77.182  2a02:c207:3013:4720:0000:0000:0000:0001  admin        
  100132643   vmd132643               running  4efbc0ba-2313-4fe1-842a-516f8652e729  EU      V45        62.171.183.4   2a02:c207:3013:2643:0000:0000:0000:0001  admin 
```

**Start Compute Instance**

`cntb start instance 12345`

**Stop Compute Instance**

`cntb stop instance 12345`
## cntb get

**List available images**

`cntb get images`

```sh
Images examples

 - 4efbc0ba-2313-4fe1-842a-516f8652e729  debian-12
 - 81d9280e-8753-40ae-8aef-7f6e20751b85  almalinux-9
 - d64d5c6c-9dda-4e38-8174-0ee282474d8a  ubuntu-24.04
 - 69b52ee3-2fda-4f44-b8de-69e480d87c7d  archlinux
 - fe6c2c36-031e-4474-aa5c-c5297196c80e  rockylinux-9
 - 1732390b-dd4a-49a4-9b40-c22e652c6df6  opensuse-leap-15.5
```

**Alle instance ausgeben**

`cntb get instances`

_Ausgabe_

```sh
  INSTANCEID  NAME       DISPLAYNAME  STATUS   IMAGEID                               REGION  PRODUCTID  IPV4           IPV6                                     DEFAULTUSER  
  100154798   vmd154798               running  4efbc0ba-2313-4fe1-842a-516f8652e729  EU      V45        86.48.5.122    2a02:c206:3015:4798:0000:0000:0000:0001  root         
  100134720   vmd134720               running  4efbc0ba-2313-4fe1-842a-516f8652e729  EU      V45        144.91.77.182  2a02:c207:3013:4720:0000:0000:0000:0001  root         
  100132643   vmd132643               running  4efbc0ba-2313-4fe1-842a-516f8652e729  EU      V45        62.171.183.4   2a02:c207:3013:2643:0000:0000:0000:0001  root         
  100047205   vmd47205                running  4efbc0ba-2313-4fe1-842a-516f8652e729  EU      V1         144.91.66.202  2a02:c207:3004:7205:0000:0000:0000:0001  root
```

**Secrets ausgeben**

```
Usage:
  cntb get secrets [flags]

Examples:
cntb get secrets

Flags:
  -h, --help          help for secrets
  -n, --name string   Filter by secret name
  -t, --type string   Filter by secret type [ssh|password]

Global Flags:
      --api string                    API base url to be used (default "https://api.contabo.com")
      --config string                 config file (Looks up /etc/cntb/.cntb.yaml then $HOME/.cntb.yaml)
  -d, --debug string                  debug level [fatal|error|warn|info|debug|trace] (default "warn")
      --oauth2-client-secret string   OAuth2 Secret. Please refer to you customer panel to retrieve this information
      --oauth2-clientid string        OAuth2 ClientId. Please refer to you customer panel to retrieve this information
      --oauth2-password string        OAuth2 User Password. Please refer to you customer panel to retrieve this information
      --oauth2-tokenurl string        OAuth2 Token URL. Please refer to you customer panel to retrieve this information (default "https://auth.contabo.com/auth/realms/contabo/protocol/openid-connect/token")
      --oauth2-user string            OAuth2 User. Please refer to you customer panel to retrieve this information
  -b, --orderBy string                Order results by these fields. E.g. name:asc. (default "name:asc")
  -o, --output string                 output format could be json|yaml|normal(=delimiter)|wide(=delimiter)|jsonpath=...|
                                        See jsonpath [http://goessner.net/articles/JsonPath/]. Delimiter defaults to horizontally aligned spaces, you could also use ',' for csv format. (default "normal")
  -p, --page int                      Page number to display. (default 1)
  -s, --size int                      Number of elements per page. (default 100)
```

`cntb get secrets`

_Ausgabe_

```sh
  SECRETID  NAME                           TYPE      
  104655    vmd134720 user root            password  
  104656    vmd134720 user root SSHPubKey  ssh
```

**User ausgeben**

`tobkern@srv3:~$ cntb get users`

_Ausgabe_

```sh
  USERID                                FIRSTNAME  LASTNAME  EMAIL                       ENABLED  
  50ba20b7-b597-491c-8c78-b9ecc578c768  Tobias     Kern      tobkern1980@gmail.com       true     
  52b392b6-c493-491d-9052-a66baf91ba4a  Tobias     Kern      tobkern1980@googlemail.com  true
```

## cntb reinstall

```sh
Usage:
  cntb reinstall instance [instanceId] [flags]

Flags:
      --defaultUser string   The default user of the instance [root, admin, administrator] (default "admin")
  -h, --help                 help for instance
      --imageId string       instance image id.
      --rootPassword int     id of stored password. User is admin with administrative/root privileges. For Linux/BSD based systems please use SSH. For Windows please use RDP.
      --sshKeys int64Slice   ids of stored SSH public keys. Applicable for Linux/BSD systems. (default [])
      --userData string      instance user data

Global Flags:
      --api string                    API base url to be used (default "https://api.contabo.com")
      --config string                 config file (Looks up /etc/cntb/.cntb.yaml then $HOME/.cntb.yaml)
  -d, --debug string                  debug level [fatal|error|warn|info|debug|trace] (default "warn")
  -f, --file string                   file or stdin (-) as input for instance reinstall. Input type might be either json or yaml.
      --oauth2-client-secret string   OAuth2 Secret. Please refer to you customer panel to retrieve this information
      --oauth2-clientid string        OAuth2 ClientId. Please refer to you customer panel to retrieve this information
      --oauth2-password string        OAuth2 User Password. Please refer to you customer panel to retrieve this information
      --oauth2-tokenurl string        OAuth2 Token URL. Please refer to you customer panel to retrieve this information (default "https://auth.contabo.com/auth/realms/contabo/protocol/openid-connect/token")
      --oauth2-user string            OAuth2 User. Please refer to you customer panel to retrieve this information
  -o, --output string                 output format could be json|yaml|normal(=delimiter)|wide(=delimiter)|jsonpath=...|
      
```

**Reinstall Instance **

`cntb reinstall instance 100134720 --defaultUser=admin --imageId=4efbc0ba-2313-4fe1-842a-516f8652e729` 

```sh
#!/bin/bash
INSTANCES=(

)

for $INSTANCE in $INSTANCES[@]
	do
	echo perform reinsrall for $INSTANCE 

#for $INSTANCE in $INSTANCES[@]
#	do
#	cntb reinstall instance $INSTANCE --defaultUser=admin --#imageId=4efbc0ba-2313-4fe1-842a-516f8652e729

```

_Ausgabe_

```sh
  INSTANCEID  NAME       DISPLAYNAME  STATUS   IMAGEID                               REGION  PRODUCTID  IPV4           IPV6                                     DEFAULTUSER  
  100132643   vmd132643               running  4efbc0ba-2313-4fe1-842a-516f8652e729  EU      V45        62.171.183.4   2a02:c207:3013:2643:0000:0000:0000:0001  root         
  100047205   vmd47205                running  4efbc0ba-2313-4fe1-842a-516f8652e729  EU      V1         144.91.66.202  2a02:c207:3004:7205:0000:0000:0000:0001  admin        
  100134720   vmd134720               running  4efbc0ba-2313-4fe1-842a-516f8652e729  EU      V45        144.91.77.182  2a02:c207:3013:4720:0000:0000:0000:0001  admin        
  100154798   vmd154798               running  
```

## cntb delete

**Delete User**

`cntb delete user 52b392b6-c493-491d-9052-a66baf91ba4a`

**Delete Screts**

```sh
cntb delete secret 104655
cntb delete secret 104656
```
