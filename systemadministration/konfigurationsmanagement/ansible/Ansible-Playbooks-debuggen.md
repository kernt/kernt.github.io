---
tags:
  - konfigurationsmanagement
  - ansible
  - dbuging
  - fehlersuche
  - fehlerbehebung
---

Wenn ein Task eines [**Ansible**](https://www.ansible.com/) Playbooks Probleme macht oder es zu einem unerwarteten Fehlverhalten kommt ist es oft nicht ganz nachvollziehbar was da passiert ist, bzw. ob z.B. auch die entsprechenden Variablen richtig gesetzt wurden.

Nachfolgend ein paar Schritte, welche bei der Fehlersuche helfen können…

### Verbosity

Um zusätzliche Informationen während eines Playbook-Laufes angezeigt zu bekommen erhöht man einfach die **verbosity**.  
Dies geschieht mit der dafür üblichen Option `-v`. Hierbei gibt die Anzahl der `v`’s das Level der Ausführlichkeit an. `-vvvv` ist die Stufe mit den ausführlichsten Informationen.

Auszug aus `ansible --help`:

```sh
-v, --verbose
verbose mode (-vvv for more, -vvvv to enable connection debugging)
```

##### Anwendung

Man kann beim Aufruf des Playbooks im Terminal die Option angeben:  
`ansible-playbook -vvvv -i hosts playbook.yml`

oder man kann auch mit Umgebungsvariablen arbeiten:  
`ANSIBLE_VERBOSITY=4 ansible-playbook -i hosts playbook.yml`

Nun wird aus der regulären Ausgabe:

```sh
PLAY [127.0.0.1] ***********************************************************************************
TASK [Hello World!] ********************************************************************************
ok: [localhost] => {
    "msg": "Hello world!"
}
PLAY RECAP *****************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0 
```

folgende:

```
ansible-playbook 2.7.8
  config file = None
  configured module search path = [u'/home/rasputin/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /home/rasputin/Ansible/ansible_2.7.8/lib/python2.7/site-packages/ansible
  executable location = /home/rasputin/Ansible/ansible_2.7.8/bin/ansible-playbook
  python version = 2.7.15 (default, Oct 15 2018, 15:26:09) [GCC 8.2.1 20180801 (Red Hat 8.2.1-2)]
No config file found; using defaults
setting up inventory plugins
/home/rasputin/github/ansible_helloworld/hosts did not meet host_list requirements, check plugin documentation if this is unexpected
/home/rasputin/github/ansible_helloworld/hosts did not meet script requirements, check plugin documentation if this is unexpected
Set default localhost to localhost
Parsed /home/rasputin/github/ansible_helloworld/hosts inventory source with ini plugin
Loading callback plugin default of type stdout, v2.0 from /home/rasputin/Ansible/ansible_2.7.8/lib/python2.7/site-packages/ansible/plugins/callback/default.pyc
PLAYBOOK: playbook.yml *****************************************************************************
1 plays in playbook.yml
PLAY [127.0.0.1] ***********************************************************************************
META: ran handlers
TASK [Hello World!] ********************************************************************************
task path: /home/rasputin/github/ansible_helloworld/playbook.yml:6
ok: [localhost] => {
    "msg": "Hello world!"
}
META: ran handlers
META: ran handlers
PLAY RECAP *****************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0   
```

### Debug Strategy

[**Strategies**](https://docs.ansible.com/ansible/latest/user_guide/playbooks_strategies.html) regeln in Ansible die Ausführung der Tasks.

Der Standardwert ist hierbei `linear`, d.h. es wird je immer ein Task auf allen entsprechenden Hosts durchgeführt und wenn dieser auf allen Systemen abgeschloßen ist geht es zum nächsten Task.

Eine weitere **strategy** die man hier verwenden kann lautet `debug`.  
Hierbei erscheint bei einem fehlgeschlagenen Task eine Debug-Konsole, in der man prüfen kann welche Variablen gerade mit welchen Werten gesetzt sind und man kann diese dann ggf. entsprechend ändern.

Dazu gibt man entweder auf der `Play`-Ebene des Playbooks den String: `strategy: debug` an:

```yaml
---
- hosts: all
  strategy: debug
...
```

oder arbeitet wieder mit Umgebungsvariablen:  
`export ANSIBLE_STRATEGY=debug`

Nachfolgend mal ein Beispiel, bei dem die Pfadangabe für eine Datei einen Tippfehler enthält und deshalb nicht erstellt werden kann.

```sh
$ ansible-playbook -i hosts playbook.yml
PLAY [all] ***********************************************************************************************************
TASK [Write my file] *************************************************************************************************
fatal: [testserver01]: FAILED! => {"changed": false, "checksum": "0ded82d0b36ca79392d9bd97debb6fbbb7bb93ef", "msg": "Destination directory /tmpp does not exist"}
[testserver01] TASK: Write my file (debug)> task.args
{u'dest': u'/tmpp/testfile', u'content': u'Hello World!\nThis is my file.\n'}
[testserver01] TASK: Write my file (debug)> task.args['dest']='/tmp/testfile'
[testserver01] TASK: Write my file (debug)> task.args
{u'dest': '/tmp/testfile', u'content': u'Hello World!\nThis is my file.\n'}
[testserver01] TASK: Write my file (debug)> redo
changed: [testserver01]
PLAY RECAP ***********************************************************************************************************
testserver01               : ok=1    changed=1    unreachable=0    failed=0   
```

Einzeln erklärt:

- `fatal: [testserver01]: FAILED! => {"changed": false, "checksum": "0ded82d0b36ca79392d9bd97debb6fbbb7bb93ef", "msg": "Destination directory /tmpp does not exist"}`  
    —> Die Fehlermeldung, welche meldet das es das entsprechende Verzeichnis nicht gibt.

- `[testserver01] TASK: Write my file (debug)> task.args`  
    `{u'dest': u'/tmpp/testfile', u'content': u'Hello World!\nThis is my file.\n'}`  
    —> Es öffnet sich die `debug`-Konsole und mit dem Befehl `task.args` kann man sich die an dieser Stelle verwendeten Variablen anzeigen lassen.  
    Alternativ: `(debug)> p task.args`  
    —> Hier sieht man auch, dass ein falscher Wert für den `dest` Parameter gesetzt ist.

- `[testserver01] TASK: Write my file (debug)> task.args['dest']='/tmp/testfile'`  
    —> Der entsprechende Eintrag im Array der Variablen wird neu gesetzt.

- `[testserver01] TASK: Write my file (debug)> task.args`  
    `{u'dest': '/tmp/testfile', u'content': u'Hello World!\nThis is my file.\n'}`  
    —> Die erneute Prüfung der variablen lässt erkennen, dass nun der richtige Pfad gesetzt ist

- `[testserver01] TASK: Write my file (debug)> redo`  
    —> Der letzte (fehlgeschlagene) Task wird mit den neuen Werten wiederholt.

Mithilfe des Debuggers lassen sich aber nicht nur bestehende Werte ändern, sondern auch neue setzen oder bestehende löschen:

```sh
(debug) task.args['owner']='myuser'
(debug) task.args
{u'dest': '/tmp/my/myfile', u'content': u'Hello World!\nThis is my file.\n', 'owner': 'myuser'}
(debug) del(task.args['owner'])
(debug) task.args
{u'dest': '/tmp/my/myfile', u'content': u'Hello World!\nThis is my file.\n'}
```

Die vom debugger verstandenen Befehle:

```sh
Documented commands (type help <topic>):
========================================
EOF  c  continue  h  help  p  pprint  q  quit  r  redo
```

### Das debugger Keyword

Das [**Debugger Keyword**](https://docs.ansible.com/ansible/latest/user_guide/playbooks_debugger.html#using-the-debugger-keyword) kann in jedem Block verwendet werden, welches ein `name` Attribut bereitstellt, z.B.: eine Rolle, ein Task, etc..  
Somit kann es zielgerichteter eingesetzt werden und es muss nicht die gesamte **strategy** geändert werden.

Das `debugger` Keyword akzeptiert folgende Werte:

- **always** – der debugger wird immer mit ausgeführt, egal was das Ergebnis ist
- **never** – der debugger wird niemals ausgeführt
- **on_failed** der debugger wird nur aufgerufen wenn ein Task fehlschlägt
- **on_unreachable** – der debugger wird aufgerufen wenn ein Host nicht erreichbar ist
- **on_skipped** – der debugger wird immer dann aufgerufen wenn ein Task übersprungen wird

In einem Task:

```yaml
- name: Execute a command
  command: false
  debugger: on_failed
```

In einem Play:

```yaml
- name: Play
  hosts: all
  debugger: never
  tasks:
    - name: Execute a command
      command: false
      debugger: on_failed
```

Hierbei sieht man gut, dass der Wert des `debugger` in einem Task den globalen Wert aus dem Play überschreibt.

Auch hierbei kann eine entsprechende Umgebungsvariable vor den Aufruf des Playbooks gesetzt werden:  
`ANSIBLE_ENABLE_TASK_DEBUGGER=True ansible-playbook -i hosts playbook.yml`

### Das `debug` Modul

Ansible hat die Möglichkeit bestimmte Ausgaben von Befehlen in eigene Variablen schreiben zu lassen. Dies geschieht mit dem Keyword `register`.  
Um sich diese und auch andere Variablen ausgeben zu lassen kann man das `debug` Modul verwenden.  
Hierbei kann man die Fülle der Ausgabe zusätzlich auch mit `verbosity` regeln:

Beispiel:

```yaml
- name: Write file
  copy:
    dest: /tmp/my
    content: |
      Hello World!
      This is my file.
    register: write_file_result
- name: Debug write_file_result
    debug:
      var: write_file_result
      verbosity: 2
```

Die entsprechende Ausgabe:

```sh
TASK [Debug write_file_result] **********************************************
ok: [localhost] => {
    "write_file_result": {
        "changed": true,
        "checksum": "0ded82d0b36ca79392d9bd97debb6fbbb7bb93ef",
        "dest": "/tmp/my/myfile",
        "gid": 20,
        "group": "staff",
        "md5sum": "f52e35e56cd4087de0948890215f7de0",
        "mode": "0644",
        "owner": "myuser",
        "size": 30,
        "src": "/Users/myuser/.ansible/tmp/ansible-tmp-1498165508.64-230947914847899/source",
        "state": "file",
        "uid": 501
    }
}
```