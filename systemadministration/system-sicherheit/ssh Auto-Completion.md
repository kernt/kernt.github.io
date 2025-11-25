Um für SSH eine Auto-Completion zu erstellen geht man wie folgt vor:

Zuerst erstellen wir uns Dateien, welche die spätere ssh-Config beinhaltet.

```
mkdir ~/.ssh/
touch ~/.ssh/config ~/.ssh/config.base
```

Nun befüllen wir **config.base** entsprechend, z.B. mit:

```
Host *
        ForwardAgent yes
```

Dann die **.bash_profile** editieren:

```
function updateSSHCompletes() {
        cat ~/.ssh/config.base > ~/.ssh/config;
        grep -v "#" /etc/hosts | grep "\." | grep -v localhost | sort | uniq | while read ip hostnames; do
                echo $hostnames | tr -s ' ' | tr ' ' '\n' | sort | uniq | while read host; do
                        echo "Host ${host}" >> ~/.ssh/config; echo -e "\tHostname ${ip}\n" >> ~/.ssh/config;
                done;
        done;
}
updateSSHCompletes
complete -o default -o nospace -W "$(grep "Host " $HOME/.ssh/config | awk '{print $2}' | grep [a-zA-Z])" scp sftp ssh
```

Hier passiert folgendes:

- .ssh/config.base wird zur neuen .ssh/config
- Hosts in /etc/hosts ausgelesen, pro Pärchen IP – Hostname wird ein Eintrag alá

```
Host test-live-db01
        Hostname 192.168.2.111
```

gefüllt

- complete-Funktion wird für scp, sftp und ssh mit den Werten aus der .ssh/config gefüttert

Jetzt kann per  
`ssh test-li<Tab><Tab>`

der Host gefunden werden.