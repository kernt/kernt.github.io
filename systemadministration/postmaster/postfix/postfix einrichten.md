Postfix ist eine beliebte und weit verbreitete Open-Source-Postübertragungsagent (MTA)-Software. Ein MTA ist verantwortlich für das Routing und die Bereitstellung von E-Mail-Nachrichten innerhalb eines Computernetzwerks, ob es sich bei diesem Netzwerk um das Internet oder ein internes Netzwerk innerhalb einer Organisation handelt.

# Anforderungen:

1. E-Mail-Anmeldeinformationen (Microsoft 365/Gmail/Outlook)
2. Linux Maschine

# **Schritt 1** - Installieren Sie die benötigten Pakete:

- **Für Redhat**    
`sudo yum install postfix cyrus-sasl-plain mailx -y`  
  
- **Für Debian Based**  
`sudo apt install postfix cyrus-sasl-plain mailx -y`

# Schritt 2 - Konfigurieren von Postfix

Öffnen Sie die **Datei /etc/postfix/main.cf** mit Ihrem Lieblingseditor (Vim/nano/) und fügen Sie die folgenden Zeilen hinzu:

vim **/etc/postfix/main.cf**

```sh
relayhost [smtp.office365.com]:587  
mynetworks > 127.0.0.0/8  
inet-interfaces - nur Loopback  
smtp-use-tls . ja  
smtp-always-send-ehloyes  
smtp-sasl-auth-enable .yes  
smtp-sasl-password-maps > Hash :/etc/postfix/sasl-passwd  
smtp-sasl-security-options - noanonym  
smtp-sasl-tls-security-options  
smtp-tls-security-level  
smtp-generic-maps - Hash :/etc/postfix/generic  
smtp-sender-unabhängige authentication .yes  
sender-dependent-relayhost-maps - Hash :/etc/postfix/sender-relay
```

# Schritt 3 Passwort

Bearbeiten der Passwortdatei **/etc/postfix/sasl-passwd**

Für Office 365 E-Mail-ID-Nutzung wie unten

**vim /etc/postfix/sasl-passwd**

```
[smtp.office365.com]:587 email_1@mail.com:PASSWORD
```

# Schritt 4-er konfigurieren mehrere E-Mails [Optional]

wenn Sie mehrere E-Mails benötigen, fügen Sie sie wie unten hinzu

**vim /etc/postfix/sasl-passwd**

```
[smtp.office365.com]:587 email_1@mail.com:PASSWORD  
[smtp.office365.com]:587 email_2@mail.com:PASSWORD  
[smtp.office365.com]:587 email_3@mail.com:PASSWORD
```

# Schritt 5- Konfigurieren Sie das Sender-Staffel [für mehrere E-Mails erforderlich]

**Bearbeiten Sie unten /etc/postfix/sender-relay** add

**vim /etc/postfix/sender-relay**

```
email_1@mail.com                         [smtp.office365.com]:587  
email_2@mail.com                            email_2@mail.com  
email_2@mail.com                            email_2@mail.com
```

# Schritt 6 ordnen Sie die Dateien nach postfix

Sobald alles fertig ist, ordnen Sie die Dateien in Postfix. Führen Sie die folgenden Befehle

```
postmap main.cf  
postmap sasl-passwd  
postmap senderer-relay
```

# Schritt 7-Mail senden

Um eine E-Mail zu senden, führen Sie den Befehl unten aus.

```
$ echo  "This is the Test Email" | mailx -r "sender_email@mail.com"  -s "This test mail " recevier_email@mail.com
```

# Schritt 8-Check-Logs [Optional]

Für Debugging-Logfile befindet sich die Datei auf **/var/log/maillog.**

Offene Logdatei: **schwanz -f /var/log/maillog**

# Verwendung von Mailx:

- -a Add-Anhang
- -b Bcc-E-Mail hinzufügen
- -c hinzufügen CC E-Mail
- -r aus E-Mail-Adresse
- -s Gegenstand der E-Mail

**Die Nachricht aus einer Datei nehmen**

`mailx -s "Test Mail by mailx" person@example.com < file.txt`

**Senden Sie dieselbe Mail an mehrere Empfänger:**

Wir können die gleiche E-Mail an mehrere Empfänger senden (nicht von cc oder bcc) wie folgt:

```bash
mailx - s "Test Mail by mailx" email_1@example.com, email_2@example.com < file.txt
```

**Hinzufügen von Anhängen**

```bash
mailx - s "Test Mail by mailx" email_1@example.com -a File.txt
```

**Hinzufügen von CC & BCC**

```bash
mail - s "A mail sent using mailx" email_0@example.com -c email_cc@example.com -b email_bcc@example.com
```

**Postfix reconfiguieren**

```sh
# Basis-Konfiguration vornehmen:
sudo dpkg-reconfigure postfix
```
