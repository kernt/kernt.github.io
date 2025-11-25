---
tags:
  - konfigurationsmanagement
  - ansible
  - auth
  - social-auth
  - oauth
---
# Authentifikation einrichten mit social Auth tools

Basierend auf [oauth](https://oauth.net/)  kann man diverse Sosial Accound nutzen um sich zusätzlich zu einem Accound zu Authentifizieren um die Sicherheit zu erhöhen.

## Google Authentifikator benutzen

den Google Authetifikator man die Sicherheit der Systeme massiv erhören.
Aber warum ist das so?
Nun ganz einfach sollte man, wie auch immer, mal jemanden sein Password verraten haben so würde es einem Angreifer immer noch nichts bringen da man noch immer etwas brauch, dass man besitzt so z.B ein Handy mit dem **Google Autheticator** ohne den kommt man immer noch nicht weiter was die Sicherheit Stark verbessert.

Hier folgt nun eine Bereinigung wie man es auch mit [Ansible](../ansible) integrieren kann, hier mit der Kostenlosen Variante.

## Zielmaschiene Einrichten

### Folgendes muss eingerichtet werden

1.Einrichten von zwei Benutzern: **ansible** und **ansible_tunnel**
2.Den eigenen public key in **~/.ssh/authorized_keys** für beide Benuzer ablegen 
3.Setzen der Shell für den Benutzer **ansible_tunnel** zu **/bin/false**, oder den user deactiviren - es wird nur benutzt für tunneling, nicht um Kommandos auszuführen.
4.Füge folgendes zur **/etc/ssh/sshd_config** hinzu:

```sh
AllowTcpForwarding no

AllowUsers ansible@127.0.0.1 ansible_tunnel

Match User ansible_tunnel
  AllowTcpForwarding yes
  PermitOpen 127.0.0.1:22
  ForceCommand echo 'This account can only be used for tunneling SSH sessions'
```

1.Setup **google-authenticator** nur für **ansible_tunnel** ausführen
1.Den **systemctl restart sshd** neu Starten

**Quelle:**

* [google-authenticator](https://www.debinux.de/2015/01/google-authenticator-als-pam-modul-im-openssh-server/)
* [Archlinux wiki Google_Authenticator](https://wiki.archlinux.org/index.php/Google_Authenticator)
* [Ansible Rolle](https://github.com/CoffeeAndCode/ansible-duo)