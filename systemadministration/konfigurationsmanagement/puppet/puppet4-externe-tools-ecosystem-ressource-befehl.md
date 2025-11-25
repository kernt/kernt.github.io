---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4 externe tools ecosystem ressource befehl

Wenn du einen Server hast, der bereits so konfiguriert ist, wie er sein muss oder fast so, kannst du diese Konfiguration als Puppenmanifest erfassen. Der Puppet-Ressource-Befehl erzeugt Puppet-Manifeste aus der vorhandenen Konfiguration eines Systems. Zum Beispiel können Sie `puppet resource` generieren ein Manifest, das alle Benutzer auf dem System gefunden erstellt. Dies ist sehr nützlich, um einen Schnappschuss eines Arbeitssystems zu machen und seine Konfiguration schnell in die Puppe zu bringen.

## Wie es geht

Hier sind einige Beispiele für die Verwendung von `puppet resource`, um Daten aus einem laufenden System zu erhalten:

1.Um das Manifest für einen bestimmten Benutzer zu generieren, führen Sie den folgenden Befehl aus:

```pp
[root@cookbook ~]# puppet resource user thomas
user { 'thomas':
  ensure           => 'present',
  comment          => 'thomas Admin User',
  gid              => '1001',
  groups           => ['bin', 'wheel'],
  home             =>../puppet/ho../puppet/thomas',
  password         => '!!',
  password_max_age => '99999',
  password_min_age => '0',
  shell            =>../puppet/b../puppet/bash',
  uid              => '1001',
}
```

2.Führen Sie für einen bestimmten Dienst den folgenden Befehl aus:

```s
[root@cookbook ~]# puppet resource service sshd
service { 'sshd':
  ensure => 'running',
  enable => 'true',
}
```

3.Führen Sie für ein Paket den folgenden Befehl aus:

```s
[root@cookbook ~]# puppet resource package kernel
package { 'kernel':
  ensure => '2.6.32-431.23.3.el6',
}
```

## Es gibt mehr

Sie können die `puppet resource` verwenden, um jede der in der Puppe verfügbaren Ressourcentypen zu untersuchen. In den vorangegangenen Beispielen haben wir ein Manifest für eine bestimmte Instanz des Ressourcentyps generiert, aber Sie können auch die `puppet resource` verwenden, um alle Instanzen der Ressource zu entleeren:

```s
[root@cookbook ~]# puppet resource service
service { 'abrt-ccpp':
  ensure => 'running',
  enable => 'true',
}
service { 'abrt-oops':
  ensure => 'running',
  enable => 'true',
}
service { 'abrtd':
  ensure => 'running',
  enable => 'true',
}
service { 'acpid':
  ensure => 'running',
  enable => 'true',
}
service { 'atd':
  ensure => 'running',
  enable => 'true',
}
service { 'auditd':
  ensure => 'running',
  enable => 'true',
}
```

Dadurch wird der Zustand jedes Dienstes auf dem System ausgegeben. Dies liegt daran, dass jeder Dienst eine aufzählbare Ressource ist. Wenn Sie den gleichen Befehl mit einer Ressource ausführen, die nicht aufzählbar ist, erhalten Sie eine Fehlermeldung:

```s
[root@cookbook ~]# puppet resource file
Error: Could not run: Listing all file instances is not supported.  Please specify a file or directory, e.g. puppet resource fil../puppet/etc
```

Asking Puppet, um jede Datei auf dem System zu beschreiben wird nicht funktionieren; Das ist etwas Bestes übrig für ein Audit-Tool wie `tripwire` (ein System entwickelt, um nach Änderungen an jeder Datei auf dem System, [Tripwire](htt../puppet//www.tripwire.com) ) zu suchen.
