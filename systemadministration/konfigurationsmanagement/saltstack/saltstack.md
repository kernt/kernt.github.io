---
tags:
  - konfigurationsmanagement
  - deployment
  - saltstack
  - salt
---
# Saltstack Allgemein 

In der Datei `/etc/salt/grains` sind eigene Variablen definiert

Begriffe:

* Minions (Zeilsysteme)
* Pillars
* Module (Befehle vom Master zu Minions)
* Slat states (Rezepte eines Ziel Zustandes von definierten Minions )
* roster

Minions sind die Zielsysteme.

Pillars sind **flexible Key-Value-Datenspeicher**. Diese bieten die Möglichkeit, Konfigurationen aus Text-Dateien auszulagern. Der gesamte Pillar-Store wird von Jinja geladen, so dass auf alle Schlüssel zugegriffen werden kann. States werden komplett zum Minion kopiert und dort von Jinja verarbeitet.

# Saltstack Installation

```sh
curl -LSs https://bootstrap.saltproject.io/ -o bootstrap-salt.sh
sudo sh bootstrap-salt.sh -x python3       # Just install the Minion
sudo sh bootstrap-salt.sh -x python3 -M -N # Just install the Master
sudo sh bootstrap-salt.sh -x python3 -M    # To install Master and Minion
```
## Saltstack HdHock Kommandos mit salt

**Saltcmd commando ausführen**

`salt '*' cmd.run "ls -l" `

**Status eines Benutzers ausgeben**

`sudo salt '*' state.sls user-jenkins`

**Service state abfragen**

`salt-all service.status apache2`

**Ping mit salt**

`salt 'vmd36612' test.ping`

**SLS Datei ausführen**

`sudo salt  $(hostname -s) state.show_sls user-jenkins`

> Es wird hier vorausgesetzt, das z.b im Verzeichnis _/srv/salt_ die `*.sls` Datein stehen. Bei diesem Beispiel ist es die Datei `user-jenkins.sls`
> Wenn keine `*.sls` für die Abfrage definiert ist sollte folgender Fehler angeziegt werden `- No matching sls found for 'user-tobkern' in env 'base'`

**Grains im json format ausgeben**

`salt 'minion-node' grains.items --out=json`

**Bestimmte Grains ausgeben**

`salt 'minion-node' grains.item id zmqversion kernel os_family --out=pprint`

**Systeme via Grain auswählen**

`salt -G 'os_family:RedHat' test.ping`

**Dokumentation finden**

`salt '*' sys.doc test.fib`

**list sys functions**

`salt '*' sys.list_functions sys`

**Inspeckt state**

`salt --verbose '*' test.sleep 2`

**salt hello**

`salt '*' cmd.run_all 'echo HELLO'`

**Grains os famely**

`salt '*' grains.item os_family`

**Armed with this information, we can target just our Debian machines using the flag
for targeting grains**

`sudo salt --grain 'os_family:Debian' test.ping`

**Or we can target just our RedHat machines, as follows:**

`sudo salt --grain 'os_family:RedHat' test.ping`

 **we could target more specifically—that is, perhaps just our Ubuntu machines**

`sudo salt -G 'os:Ubuntu' test.ping`

**list users**

`salt '*' user.list_users`

## Masterless salt call

**Service neu Starten**

`sudo salt-call service.restart selenium-node`

## Saltstack Compound matching

**compound match can look like this**

`sudo salt -C '*minion and G@os:Ubuntu and not L@yourminion,theirminion'`

`sudo salt -C '* and not G@os_family:RedHat' test.ping`

`salt -C 'G@os:Ubuntu or G@os:Fedora' test.ping`

## Installing packages

`salt '*' sys.doc pkg.install`

`salt '*' pkg.install apache2`

`salt '*' pkg.list_pkgs`

`salt '*' pkg.version htop`
## Remote execution modules and functions

`salt '*' sys.list_modules`

`salt '*' sys.doc user.add`

`salt '*' user.info larry`
## cmd

`salt '*' cmd.run_stdout 'echo Hello!'`

`salt '*' cmd.run_stderr 'echo Hello!'`

`salt '*' cmd.retcode 'echo Hello!'`

`salt '*' cmd.run_all 'echo Hello!'`

## Saltstack neues Rezept erstellen


## Saltstack sls Datien


**/srv/salt/install.sls**

```sh
utilitler:
  pkg.installed:
    {% if grains[‘os’] == ‘CentOS‘ %}
    – name: nload
    {% elif grains[‘os’] == ‘Ubuntu‘ %}
    – name: nload
    {% elif grains[‘os’] == ‘FreeBSD‘ %}
    – name: nload
    {% endif %}

```
## Saltstack Debugging

`salt-call --local pkg.group_list -l debug`

## Eigene Grains Definieren

Ein Minimalbeispiel eigener grains ist, in /etc/salt/grains ein key: value-Paar zu setzen:

```sh
sed 's/^[ ]*//' <<EOF | sudo tee -a /etc/salt/grains
SLA: 24/7-Economie
EOF
```

Danach sollte mit:

`salt -G 'SLA:24/7-Economie' test.ping --out=pprint`

die passenden Systeme selektierbar sein.

In der Praxis werden gerne mehrere werte für eine Variable zu setzen:

`sudo salt 'minion-node' grains.setval roles  "['server', 'developer']"`

Vorausgsetzt wir hätten ein Variable `sla: platinium247` defeniert

Mit folgender Ausgabe:

```sh
{
    "vmd36612": {
        "id": "vmd36612",
        "sla": "platinium247"
    }
}
````

Und möchten nun z.b mit Python den ausgabe parsen.
Dazu können wir die json wert von `sla` auswerten.
Bei mehren werten muss man nur Überlegen wie weit man in der strucktur vorwertz gehen muss um auf die Ebene zum Wert zu kommen

```sh
salt 'vmd36612' grains.item id  sla --out=json | python -c 'import sys, json; print json.load(sys.stdin)["vmd36612"]["sla"]'
```

Wenn wir z.b nun unser Beispile erweitern

```sh
{
    "vmd36612": {
        "id": "vmd36612",
        "sla": "platinium247"
        "New-Cloud-Systems": {
            Platform: "aws"
            Storage: "caph"
        }
    }
}
````

## Saltstack Pillars verwenden

Da grains allen Minins zugänglich sind sollte diese nicht für z.B Benutzerpasswörter oder Lizenz Schlüssel Verwendung finden.
Dafür gibt es die Pillers bei Saltstack.

Pillers werden für 

# Saltstck modules

[augeas](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.augeas_cfg.html#module-salt.modules.augeas_cfg)
[cron](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.cron.html#module-salt.modules.cron)
[cpan](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.cpan.html#module-salt.modules.cpan)
[deb Apache](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.deb_apache.html#module-salt.modules.deb_apache)
[Postgres deb](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.deb_postgres.html#module-salt.modules.deb_postgres)
[dockercompose](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.dockercompose.html#module-salt.modules.dockercompose)
[dockermod](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.dockermod.html#module-salt.modules.dockermod)
[firewalld](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.firewalld.html#module-salt.modules.firewalld)
[extfs](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.extfs.html#module-salt.modules.extfs)
[event](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.event.html#module-salt.modules.event)
[aptly](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.aptly.html#module-salt.modules.aptly)
[aptpkg](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.aptpkg.html#module-salt.modules.aptpkg)
[git](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.git.html#module-salt.modules.git)
[github](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.github.html#module-salt.modules.github)
[grafana4](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.grafana4.html#module-salt.modules.grafana4)
[haproxy](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.haproxyconn.html#module-salt.modules.haproxyconn)
[helm](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.helm.html#module-salt.modules.helm)
[icinga2](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.icinga2.html#module-salt.modules.icinga2)
[ipset](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.ipset.html#module-salt.modules.ipset)
[iptables](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.iptables.html#module-salt.modules.iptables)
[k8s](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.k8s.html#module-salt.modules.k8s)
[Kernel APT](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.kernelpkg_linux_apt.html#module-salt.modules.kernelpkg_linux_apt)
[Kernel YUM](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.kernelpkg_linux_yum.html#module-salt.modules.kernelpkg_linux_yum)
[ldap3](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.ldap3.html#module-salt.modules.ldap3)
[ldapmod](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.ldapmod.html#module-salt.modules.ldapmod)
[linux acl](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.linux_acl.html#module-salt.modules.linux_acl)
[Linux IP](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.linux_ip.html#module-salt.modules.linux_ip)
[lvm](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.linux_lvm.html#module-salt.modules.linux_lvm)
[lvs](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.lvs.html#module-salt.modules.lvs)
[mysql](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.mysql.html#module-salt.modules.mysql)
[netaddress](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.netaddress.html#module-salt.modules.netaddress)
[network](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.network.html#module-salt.modules.network)
[NFS](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.nfs3.html#module-salt.modules.nfs3)
[nftables](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.nftables.html#module-salt.modules.nftables)
[nginx](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.nginx.html#module-salt.modules.nginx)
[osquery](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.osquery.html#module-salt.modules.osquery)
[PCS](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.pcs.html#module-salt.modules.pcs)
[parted](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.parted_partition.html#module-salt.modules.parted_partition)
[POstfix](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.postfix.html#module-salt.modules.postfix)
[PS](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.ps.html#module-salt.modules.ps)
[pyenv](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.pyenv.html#module-salt.modules.pyenv)
[s3](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.s3.html#module-salt.modules.s3)
[selinux](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.selinux.html#module-salt.modules.selinux)

## Saltstack

## Saltstck module

## Saltstack master of masters Einrichten

Master of Master ist eine Möglichkeit die Server hochverfügbar zu halten.
Alternativ kann aber auch der dezentrale Ansatz umgesetzt werden.

### Quellen

* [Automatisierung mit SaltStack](https://www.informatik-aktuell.de/entwicklung/methoden/gut-gewuerzt-automatisierung-mit-saltstack.html)
* [saltstack](https://thorstenkramm.gitbook.io/saltstack/)
* [saltstack-examples](https://www.unixmen.com/saltstack-examples/)
* [saltstack examples 2](https://adamtheautomator.com/saltstack-examples/)
- [https://github.com/rackspace-orchestration-templates/salt-states/tree/master](rackspace-orchestration-templates)