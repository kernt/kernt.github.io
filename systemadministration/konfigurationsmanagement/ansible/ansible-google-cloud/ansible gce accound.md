# Account mit Berechtigungen anlegen

- `auth_kind`: Art der Authentifizierung (Wahlen: machineaccount, serviceaccount, Anwendung)
- `service_account_email`: E-Mail im Zusammenhang mit dem Projekt
- `service_account_file`: Pfad zur JSON-Anmeldedatei
- `project`: id des Projekts
- `scopes`: Die spezifischen Zielfernrohre, die Sie von den Aktionen wollen.

**gcp_compute_address modul benutzen**

```yaml
- name: Create IP address
  hosts: localhost
  gather_facts: false
  vars:
    service_account_file: /home/my_account.json
    project: my-project
    auth_kind: serviceaccount
    scopes:
      - https://www.googleapis.com/auth/compute
  tasks:
   - name: Allocate an IP Address
     gcp_compute_address:
         state: present
         name: 'test-address1'
         region: 'us-west1'
         project: "{{ project }}"
         auth_kind: "{{ auth_kind }}"
         service_account_file: "{{ service_account_file }}"
         scopes: "{{ scopes }}"
```

**Bereitstellung von Anmeldeinformationen als Umgebungsvariablen**

```sh
GCP_AUTH_KIND
GCP_SERVICE_ACCOUNT_EMAIL
GCP_SERVICE_ACCOUNT_FILE
GCP_SCOPES
```

## GCE Dynamic Inventory

**GCE-Inventar-Plugin activiren**

```ini
[inventory]
enable_plugins = gcp_compute
```

Die Datei `.gcp.yml` im root Verzeichnis erstellen 

```
plugin: gcp_compute
projects:
  - graphite-playground
auth_kind: serviceaccount
service_account_file: /home/ansible/my_account.json
```