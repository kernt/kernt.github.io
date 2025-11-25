Nachfolgend zeige ich wie man in Ansible seine Hosts nach `gerade` bzw. `ungrade` aufteilen kann.  
Angenommen man hat sechs Webserver (z.B.: **web01** – **web06**) und möchte Tasks nur auf allen geraden bzw. ungeraden Hosts ausführen:

# Aufteilen der Hosts

```
- name: "split hosts in two groups"
  hosts: all
  gather_facts: false
  tasks:
  - group_by:
      key: "{{ 'even' if ((inventory_hostname.split('-')[2][-2:] | int) is even) else 'odd' }}"
...
  - assert:
      that:
        - groups.odd is defined
        - groups.even is defined
      msg: "gruppen konnten nicht in even/odd aufgeteilt werden"
```

Damit werden die Hosts in `even` (gerade) und `odd` (ungerade) aufgeteilt. Natürlich kann man die Gruppen auch anders benennen.  
Anschließend wird noch überprüft ob die Aufteilung funktioniert hat.

# Ausführen von Tasks auf geraden/ungeraden Hosts

Ansprechen der entsprechenden Gruppe für die Tasks geschieht per `hosts: all:&<GRUPPENNAME>`

**ungerade Hosts:**

```
- name: "deploy on odd hosts"
  hosts: all:&odd
  gather_facts: false
  tasks:
  - debug:
      msg: "odd: {{inventory_hostname}}"
```

**gerade Hosts:**

```
- name: "deploy on even hosts"
  hosts: all:&even
  gather_facts: false
  tasks:
  - debug:
      msg: "even: {{inventory_hostname}}"
```