
/var/snap/gitea/common/conf 



> Git erros on snap based install 
 unable to access '/etc/gitconfig': Permission denied


```
sudo chmod 0766 /etc/gitconfig
sudo exho "/etc/gitconfig r," > /var/lib/snapd/apparmor/profiles/snap.gitea.web

sudo apparmor_parser -r /var/lib/snapd/apparmor/profiles/snap.gitea.web

sudo snap restart gitea

```

