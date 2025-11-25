---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4-applikationen

Ohne Anwendungen ist ein Server nur eine sehr teure Raumheizung. In diesem Kapitel werde ich einige Rezepte vorstellen, um eine bestimmte Software mit Puppet zu verwalten: MySQL, Apache, Nginx und Ruby. Ich hoffe die Rezepte werden dir in sich selbst nützlich sein. Allerdings sind die Muster und Techniken, die sie verwenden, auf fast jede Software anwendbar, so dass Sie sie an Ihre eigenen Zwecke ohne viel Schwierigkeiten anpassen können. Eine Sache, die über diese Anwendungen üblich ist, sind sie üblich. Die meisten Puppeninstallationen müssen mit einem Webserver, Apache oder Nginx umgehen. Die meisten, wenn nicht alle, haben Datenbanken und einige von denen haben MySQL. Wenn jeder mit einem Problem umgehen muss, sind Community-Lösungen in der Regel besser getestet und gründlicher als homegrown Lösungen. Wir verwenden Module aus dem Puppet Forge in diesem Kapitel, um diese Anwendungen zu verwalten.

Wenn du deine eigenen Apache- oder Nginx-Module von Grund auf schreibst, musst du auf die Nuancen der Distributionen achten, die du unterstütztest. Einige Distributionen rufen das Apache-Paket `httpd` auf, während andere `Apache2` verwenden; Das gleiche gilt für MySQL. Darüber hinaus verwenden Debian-basierte Distributionen eine aktivierte Ordner-Methode, um benutzerdefinierte Websites in Apache zu aktivieren, die virtuelle Websites sind, während RPM-basierte Distributionen nicht. Weitere Informationen zu [virtuellen Websites](htt../puppet/httpd.apache.o../puppet/do../puppet/2../puppet/vhos../puppet/).

Übersicht

* [Mit öffentlichen Modulen](../puppet/puppet/puppet-applikations-public)
* [Verwalten von Apache-Servern](../puppet/puppet/puppet-applikations-apache)
* [Erstellen virtueller Host-Hosts](../puppet/puppet/puppet-applikations-virtual-hosts)
* [Erstellen von nginx virtuellen Hosts](../puppet/puppet/puppet-applikations-virtual-hosts-nginx)
* [Verwalten von MySQL](../puppet/puppet/puppet-applikations-mysql)
* [Erstellen von Datenbanken und Benutzern](../puppet/puppet/puppet-applikations-db-benutzer)