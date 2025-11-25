---
tags:
  - monitoring
  - puppet
  - konfigurationsmanagement
---
# puppet-monitoring-reporting-inspection

Sie wissen wahrscheinlich, dass die Konfigurationseinstellungen von Puppet in puppet.conf gespeichert sind, aber es gibt viele Parameter, und diejenigen, die nicht in der puppet.conf aufgeführt sind, nehmen einen Standardwert an. Wie sehen Sie den Wert eines beliebigen Konfigurationsparameters, unabhängig davon, ob es explizit in puppet.conf gesetzt ist oder nicht? Die Antwort ist, den Befehl puppet config print zu verwenden.

## Wie es geht

Führen Sie den folgenden Befehl aus. Dies wird eine Menge von Ausgabe (es kann hilfreich sein, um es durch weniger, wenn Sie möchten, um die verfügbaren Konfigurationseinstellungen zu durchsuchen):

```d
[root@cookbook ~]# puppet config print |head -25
report_serialization_format = pson
hostcsr ../puppet/v../puppet/l../puppet/pupp../puppet/s../puppet/csr_cookbook.example.com.pem
filetimeout = 15
masterhttplog ../puppet/v../puppet/l../puppet/pupp../puppet/masterhttp.log
pluginsignore = .svn CVS .git
ldapclassattrs = puppetclass
certdir ../puppet/v../puppet/l../puppet/pupp../puppet/s../puppet/certs
ignoreschedules = false
disable_per_environment_manifest = false
archive_files = false
hiera_config ../puppet/e../puppet/pupp../puppet/hiera.yaml
req_bits = 4096
clientyamldir ../puppet/v../puppet/l../puppet/pupp../puppet/client_yaml
evaltrace = false
module_working_dir ../puppet/v../puppet/l../puppet/pupp../puppet/puppet-module
tags =
cacrl ../puppet/v../puppet/l../puppet/pupp../puppet/s../puppet/../puppet/ca_crl.pem
manifest ../puppet/e../puppet/pupp../puppet/manifes../puppet/site.pp
inventory_port = 8140
ignoreimport = false
dbuser = puppet
postrun_command =
document_all = false
splaylimit = 1800
certificate_expire_warning = 5184000
```

## Wie es funktioniert

Running `puppet config print` gibt jeden Konfigurationsparameter und seinen aktuellen Wert aus (und es gibt viele davon).

Um den Wert für einen bestimmten Parameter zu sehen, füge ihn als Argument zu `puppet config print` Befehl hinzu:

```s
[root@cookbook ~]# puppet config print modulepath
/e../puppet/pupp../puppet/module../puppet/u../puppet/sha../puppet/pupp../puppet/modules
```
