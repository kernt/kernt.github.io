---
tags:
  - konfigurationsmanagement
  - ansible
  - sicherheit
  - vault
---
# Ansible Sicherheit

**Einen Benutzer daran hindern den Inhalt einen Playbooks zu sehen**

Hier wird der User daran gehindert die Variable ssh_port zu sehen und ihren Wert

**Playbook Beispiel**

```yaml
- hosts: '*'
  vars:
    ssh_port: 2049
  tasks:
    - name: Tell SELinux about SSH's New Port
      seport:
        ports: "{{ ssh_port }}"
        proto: tcp
        setype: ssh_port_t
        state: present

    - name: Harden sshd configuration
      lineinfile:
        dest: /etc/ssh/sshd_config
        regexp: "{{item.regexp}}"
        line: "{{item.line}}"
        state: present
        validate: 'sshd -T -f %s'
      with_items:
        - regexp: "^Port"
          line: "Port {{ ssh_port }}"
        - regexp: "^PermitRootLogin"
          line: "PermitRootLogin no"
        - regexp: "^AllowUsers"
          line: "AllowUsers ansible-devops"
        - regexp: "^PasswordAuthentication"
          line: "PasswordAuthentication no"
        - regexp: "^AllowAgentForwarding"
          line: "AllowAgentForwarding no"
        - regexp: "^AllowTcpForwarding"
          line: "AllowTcpForwarding no"
        - regexp: "^MaxAuthTries"
          line: "MaxAuthTries 3"
        - regexp: "^MaxSessions"
          line: "MaxSessions 6"
        - regexp: "^TCPKeepAlive"
          line: "TCPKeepAlive no"
        - regexp: "^UseDNS"
          line: "UseDNS no"
      notify: restart sshd
    
    - name: add user ansible-devops
      user:
        name: ansible-devops
    
    - name: add sudo group rights for deployment user
      lineinfile:
        dest: /etc/sudoers.d/ansible-devops
        regexp: "^ansible-devops"
        line: "ansible-devops ALL=(ALL) NOPASSWD: ALL"
        state: present
  handlers:
    - name: restart sshd
      service:
        name: sshd
        state: restarted
```

**Validieren mit yamllint**

```shell
yamllint ssh-config.yaml || echo "Success"
```

Um einen Wert in einem Playbook  zu verschlüsseln, die gewünschte Zeichenfolge angeben ( `2049`in diesem Beispiel) zusammen mit dem Schlüssel, zu dem es gehört (`ssh_port`, in diesem Beispiel). Verwenden Sie die `--ask-vault-pass`Option zur Erstellung eines Passworts aufgefordert werden. Die Ausgabe ist sehr lang, also habe ich sie wegen Klarheit abgekützt.

```shell
ansible-vault encrypt_string --ask-vault-pass '2049' --name 'ssh_port'
New vault password:
Confirm password:
ssh_port: !vault | 
          $ANSIBLE_VAULT;1.1;AES256 
          3433313631373[...]3631 
Encryption successful
```

Kopieren Sie nun das Ergebnis in Ihr Playbook. Sie müssen alles vom Schlüsselnamen kopieren (`ssh_port`) bis zum Ende der langen Zahlenkette, die die verschlüsselten Daten enthält. Es sieht ein wenig chaotisch aus, aber am Ende enthält Ihr Playbook dies (dieses Spielbuch ist für Kürze abkürzt):

```yaml
---
- hosts: '*'
  vars:
    ssh_port: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          656331383031333630343337346537343832353535643932643265323764333361373632633538376233663762623833393432653438616263396565316365330a376630353530643434653539323834633866373664623865356365366130323765396336616534626364616130376361313437366235616137663533333432390a3735636538373861656666333964643435653037666537386563613632373234
  tasks:
    - name: Tell SELinux about SSH's New Port
      seport:
        ports: "{{ ssh_port }}"
        proto: tcp
        setype: ssh_port_t
        state: present
[...]
```

Prüfung mit `yamllint`:

```shell
yamllint ssh-config.yaml || echo "Success"
Success
```

## Führen Sie ein verschlüsseltes Spielbuch aus

Um ein Spielbuch mit einem verschlüsselten String zu führen, verwenden Sie die `ansible-playbook`Befehl, Hinzufügen der `--ask-vault-pass`Option. In diesem Beispiel können Sie die Warnungen vor gültigen Hosts ignorieren, weil Sie nur ein Beispielspielbuch testen:

```
ansible-playbook --ask-vault-pass ssh-config.yaml
Vault password:  

PLAY [ssh_server] ************************************** 

TASK [Gathering Facts] ********************************** 
ok: [localhost]

TASK [Tell SELinux about SSH's New Port] **************** 
ok: [localhost]

PLAY RECAP ********************************************** 
localhost: ok=2  [...] failed=0    skipped=0
Success!
```
## Die Password Eingabe automatisieren

Beispiel einer Passwortdatei namens `secrets.txt` die folgendes  Passwort enthält:

`password123`

Um das Password zu übergeben muss man das Playbook wie folgt aufrufen:

```shell
$ ansible-playbook ssh-config.yaml --vault-id ssh_port@secrets.txt
```

Sie können auch mehrere Passwörter verwenden, indem Sie mehre Vault ID's angeben.
Hier ist eine Beispiel-Passwortdatei mit mehr als einem Passwort, vorausgesetzt, dass beide `ssh_port`und `setype`Schlüssel im Beispiel YAML-Datei sind verschlüsselt mit:

```sh
ansible-vault encrypt_string --vault-id ssh_port@secrets.txt '2049' --name 'ssh_port' 
```

und

```sh
ansible-vault encrypt_string --vault-id setype@secrets.txt 'ssh_port_t' --name 'setype'
```

bzw.

Hier ist eine Beispiel Datei `secrets.txt`

```sh
ssh_priv_key_myservers: password123
user_ansible: password456
```

Um das Playbook auszuführen

```shell
ansible-playbook --vault-id ssh_port@secrets.txt \
--vault-id setype@secrets.txt ssh-config.yaml
```

# Ansible Vault

Mit der Vault können Sensible Daten geschützt werden

**Playbook starten wen es vault-encrypted daten enthält**
`ansible-playbook site.yml --ask-vault-pass`

##### vault-id

Seit der Ansible Version 2.4 wird angeraten anstatt `--vault-password-file=<Pfad zur Datei>`  
lieber  
`--vault-id /Pfad/zur/vault-passwort-datei`  
zu verwenden. 

Möchte man das Passwort zur Entschlüsselung lieber im Terminal eingeben verwendet man:

```
--vault-id @prompt
```

Des weiteren kann auch mit Labels gearbeitet werden:

```
--vault-id dev@dev-password
```

Um bei der Verschlüsselung ein Label mit anzugeben verwendet man:

```
ansible-vault encrypt --vault-id dev@password 'foooodev' --name 'the_dev_secret' <name>.yml
```

Es können mit vault-id auch mehrere Passwörter angegeben werden.

```
ansible-playbook --vault-id dev@dev-password --vault-id prod@prompt <name>.yml
```

### Verschlüsseln eines Strings

Möchte man nicht das gesamte Playbook verschlüsseln sondern nur die sensiblen Daten, so kann man dies mit der Option `encrypt_string` realisieren.

```
ansible-vault encrypt_string 'das sind sensible Daten!' --name 'ein_string'
New Vault password: 
Confirm New Vault password: 
ein_string: !vault |
          $ANSIBLE_VAULT;1.1;AES256
346635393033326364376465353836303932646632326661353836323362656633353631666430663631363433343533366232303131343664646232636662340a316661303038343266353861636137356336303934303930636531396335653739373163633562323430643038356466653438613939643135336539613339340a313965336430313461653933623632343665646439616663323462356139
64663335333661623736616438336431363463623365356439306637356438613162
Encryption successful
```

Dies kann man nun an die entsprechende Stelle in das Playbook einsetzen.

### Verwenden von Passwort-Hashes

Man kann beim anlegen des Benutzer Passwortes auch direkt einen Hash angeben. Auf die Weise lässt sich das Passwort auch unkenntlich machen.

Um den Hash für das Passwort zu erzeugen verwendet man das Tool `mkpasswd`, welches unter Debian/Ubuntu z.B. im Paket `whois` enthalten ist.

```
$ mkpasswd --method=SHA-512
Passwort: 
$6$zrRxxO/BP$b3q6nsvVVzyKJ3DmFKcM2QVjOL2nhnB3ks74CU3yyyawpIXN93fqwQBUxewnvsr7BkH0MQifLc6fher3DDFke0
```

Diesen Wert kann man nun direkt als Variable in die entsprechenden Ansible-Dateien eintragen:

```
$ cat vars.yml
---
admin_password: $6$zrRxxO/BP$b3q6nsvVVzyKJ3DmFKcM2QVjOL2nhnB3ks74CU3yyyawpIXN93fqwQBUxewnvsr7BkH0MQifLc6fher3DDFke0
deploy_password: $6$FZ1qePd8qaK8Hd$a4jz8qIr2ekKAaDbKN8eT/UZEPWd9XBwtVLOC9ppYUkGUirg5qUcneO6RTZUJxe1xX/Pw2atLePy
```

**Quellen:**

* [how-to-use-vault-to-protect-sensitive-ansible-data](https://www.digitalocean.com/community/tutorials/how-to-use-vault-to-protect-sensitive-ansible-data-on-ubuntu-16-04)

https://raw.githubusercontent.com/ansible-community/contrib-scripts/refs/heads/main/vault/vault-keyring-client.py