---
tags:
  - konfigurationsmanagement
  - ansible
  - ansible-pull
---

# ansible-pull

Ansible bietet neben der regulären `push` Methode auch die Möglichkeit an, dass ein System sich, gesteuert durch z.b. per `cron` oder `systemd-timer`, selbstständig ein Playbook von z.B. github.com lädt und lokal ausführt.

Ein Anwendungsfall wäre z.b. die Erstellung eines Systemes, wo bei der Einrichtung bereits Ansible installiert wird und dieses dann ein entsprechendes Playbook herunterlädt und durchläuft um die Installation abzuschließen.  
Oder auch ein regelmäßiges aktualisieren verschiedener Konfigurationen oder Systemkomponenten ist damit gut realisierbar.  
Desweiteren skaliert diese Methode sehr gut.


Die entsprechende Funktion um Playbooks auf das System zu laden lautet `ansible-pull` und kommt bei der Installation von Ansible automatisch mit.

Dokumentation: [ansible-pull](https://docs.ansible.com/ansible/latest/cli/ansible-pull.html)

Als praktisches Beispiel verwenden wir ein Repo, welches nur eine `localhost.yml` mit folgendem Inhalt beinhaltet:

```
---
- name: "test"
  hosts: localhost
  tasks:
    - name: "Gib String aus"
      debug:
        msg: "Das ist ein test"
```

Nun rufen wir `ansible-pull` mit dem Parameter `-U <Repo-URL>` auf:

```
$ ansible-pull -U https://github.com/techgoat-net/ansible_pull_test.git
Starting Ansible Pull at 2019-10-23 20:29:47
/bin/ansible-pull -U https://github.com/techgoat-net/ansible_pull_test.git
 [WARNING]: Could not match supplied host pattern, ignoring: localhost.localdomain
localhost  [WARNING]| SUCCESS : Your git=> {
    " version iafter": "es too old 3d27bc760eto fully s916c9e5fe0upport thec37552c32c depth argfc397de01"ument. Fal, 
    "beling back fore": "e3to full chd27bc760e9eckouts.
16c9e5fe0c37552c32cfc397de01", 
    "changed": false, 
    "remote_url_changed": false
}
 [WARNING]: Could not match supplied host pattern, ignoring: all
 [WARNING]: provided hosts list is empty, only localhost is available
 [WARNING]: Could not match supplied host pattern, ignoring: localhost.localdomain

PLAY [test] ***************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************
ok: [localhost]

TASK [Gib String aus] *****************************************************************************************************
ok: [localhost] => {
    "msg": "Das ist ein test"
}

PLAY RECAP ****************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0   
```

`ansible-pull --accept-host-key --url  --directory /home/tobkern/git --ask-become-pass --only-if-changed`

Es gibt noch viele weitere Parameter die man hierbei angeben kann. Welche es gibt erfährt man mit `ansible-pull -h`.  
Diese Methode eignet sich auch gut für [continuous integration.](https://de.wikipedia.org/wiki/Kontinuierliche_Integration)

https://github.com/wdhowe/ansible-pull