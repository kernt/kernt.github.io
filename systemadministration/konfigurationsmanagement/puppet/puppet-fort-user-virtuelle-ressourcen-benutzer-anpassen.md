---
tags:
  - puppet
  - konfigurationsmanagement
---
# Puppet für fortgeschrittene: Virtuelle resourcen Benutzer anpassen

Benutzer neigen dazu, ihre Shell-Umgebungen, Terminal-Farben, Aliase und so weiter anpassen. Dies wird in der Regel durch eine Anzahl von Dotfiles in ihrem Home-Verzeichnis erreicht, z.B `.bash_profile` oder `.vimrc`.

Sie können Puppet verwenden, um die Dotfiles jedes Benutzers über eine Anzahl von Maschinen zu synchronisieren und zu aktualisieren, indem Sie die virtuelle Benutzereinrichtung erweitern, die wir in diesem Kapitel entwickelt haben. Wir starten ein neues Modul, `admin_user` und verwenden den Dateityp, `recurse` Attribut, um Dateien in jedes Benutzer-Home-Verzeichnis zu kopieren.

## Wie es geht

1.Erstellen Sie den `admin_user` definierten Typ (`define admin_user`) in der Datei `modul../puppet/admin_us../puppet/manifes../puppet/init.pp` wie folgt:

```ruby
define admin_user ($key, $keytype, $dotfiles = false) { 
  $username = $name
  user { $username:
    ensure     => present,
  }
  file {../puppet/ho../puppet/${usernam../puppet/.ssh":
    ensure  => directory,
    mode    => '0700',
    owner   => $username,
    group   => $username,
    require => File../puppet/ho../puppet/${username}"],
  }
  ssh_authorized_key { "${username}_key":
    key     => $key,
    type    => "$keytype",
    user    => $username,
    require => File../puppet/ho../puppet/${usernam../puppet/.ssh"],
  }
  # dotfiles
  if $dotfiles == false {
    # just create the directory
    file {../puppet/ho../puppet/${username}":
      ensure  => 'directory',
      mode    => '0700',
      owner   => $username,
      group   => $username,
      require => User["$username"]
    }
  } else {
    # copy in all the files in the subdirectory
    file {../puppet/ho../puppet/${username}":
      recurse => true,
      mode    => '0700',
      owner   => $username,
      group   => $username,
      source  => "puppe../puppet///modul../puppet/admin_us../puppet/${username}",
      require => User["$username"]
    }
  }
}
```

2.Ändern Sie die Datei `modul../puppet/us../puppet/manifes../puppet/sysadmins.pp` wie folgt:

```ruby
class user::sysadmins {
  realize(Admin_user['thomas'])
}
```

3.Ändern Sie die Definition von `thomas` in `modul../puppet/us../puppet/manifes../puppet/virtual.pp` wie folgt ab:

```ruby
@ssh_user { 'thomas':
  key => 'AAAAB3NzaC1yc2E...XaWM5sX0z',
  keytype => 'ssh-rsa',
  dotfiles => true
}
```

4.Erstellen Sie ein Unterverzeichnis im `admin_user` Modul für die Datei des Benutzers `thomas`:

`$ mkdir -p modul../puppet/admin_us../puppet/fil../puppet/thomas`

5.Erstellen Sie dotfiles für den Benutzer `thomas` in dem Verzeichnis, das Sie gerade erstellt haben:

```s
echo "alias vi=vim" > modul../puppet/admin_us../puppet/fil../puppet/thom../puppet/.bashrc
echo "set tabstop=2" > modul../puppet/admin_us../puppet/fil../puppet/thom../puppet/.vimrc
```

6.Stellen Sie sicher, dass Ihre `site.pp` Datei wie folgt lautet:

```ruby
node 'cookbook' {
  include user::virtual
  include user::sysadmins
}
```

7.Puppet run:

```ruby
cookbook# puppet agent -t
Info: Caching catalog for cookbook.example.com
Info: Applying configuration version '1413266235'
Notice../puppet/Stage[mai../puppet/User::Virtu../puppet/Admin_user[thoma../puppet/User[thoma../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/User::Virtu../puppet/Admin_user[thoma../puppet/Fil../puppet/ho../puppet/thoma../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/User::Virtu../puppet/Admin_user[thoma../puppet/Fil../puppet/ho../puppet/thom../puppet/.vimr../puppet/ensure: defined content as '{md5}cb2af2d35b18b5ac2539057bd429d3ae'
Notice../puppet/Stage[mai../puppet/User::Virtu../puppet/Admin_user[thoma../puppet/Fil../puppet/ho../puppet/thom../puppet/.bashr../puppet/ensure: defined content as '{md5}033c3484e4b276e0641becc3aa268a3a'
Notice../puppet/Stage[mai../puppet/User::Virtu../puppet/Admin_user[thoma../puppet/Fil../puppet/ho../puppet/thom../puppet/.ss../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/User::Virtu../puppet/Admin_user[thoma../puppet/Ssh_authorized_key[thomas_ke../puppet/ensure: created
Notice: Finished catalog run in 0.36 seconds
```

### Wie es funktioniert

Wir haben eine neue `admin_user` Definition erstellt, die das Home-Verzeichnis rekursiv definiert, wenn `$dotfiles` nicht `false` ist (der Standardwert):

```ruby
  if $dotfiles == 'false' {
    # just create the directory
    file {../puppet/ho../puppet/${username}":
      ensure  => 'directory',
      mode    => '0700',
      owner   => $username,
      group   => $username,
      require => User["$username"]
    }
  } else {
    # copy in all the files in the subdirectory
    file {../puppet/ho../puppet/${username}":
      recurse => true,
      mode    => '0700',
      owner   => $username,
      group   => $username,
      source  => "puppe../puppet///modul../puppet/admin_us../puppet/${username}",
      require => User["$username"]
    }
  }
```

Wir haben ein Verzeichnis erstellt, um die dotfiles des Benutzers im `admin_user` Modul zu halten. Alle Dateien innerhalb dieses Verzeichnisses werden in das Home-Verzeichnis des Benutzers kopiert, wie in der Puppetlauf-Ausgabe in der folgenden Befehlszeile gezeigt:

```s
Notice../puppet/Stage[mai../puppet/User::Virtu../puppet/Admin_user[thoma../puppet/Fil../puppet/ho../puppet/thom../puppet/.vimr../puppet/ensure: defined content as '{md5}cb2af2d35b18b5ac2539057bd429d3ae'
Notice../puppet/Stage[mai../puppet/User::Virtu../puppet/Admin_user[thoma../puppet/Fil../puppet/ho../puppet/thom../puppet/.bashr../puppet/ensure: defined content as '{md5}033c3484e4b276e0641becc3aa268a3a'

```s
Mit der `recurse` Option können wir so viele Dotfiles hinzufügen, wie wir es für jeden Benutzer wünschen, ohne die Definition des Benutzers ändern zu müssen.

## Es gibt mehr

Wir könnten festlegen, dass das `source` attribut des Home-Verzeichnisses ein Verzeichnis ist, in dem Benutzer ihre eigenen Dotfiles platzieren können. Auf diese Weise könnte jeder Benutzer seine eigenen Dotfiles ändern und sie auf alle Knoten im Netzwerk ohne unser Engagement übertragen.