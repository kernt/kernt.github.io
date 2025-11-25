# Einrichtung unter Debian/ubuntu

**Packete installieren**

`sudo aptitude install dovecot-core dovecot-lmtpd dovecot-imapd dovecot-pop3d`

## [Configure Dovecot](https://ubuntu.com/server/docs/install-and-configure-dovecot#configure-dovecot)

To configure Dovecot, edit the file `/etc/dovecot/dovecot.conf` and its included config files in `/etc/dovecot/conf.d/`. By default, all installed protocols will be enabled via an _include_ directive in `/etc/dovecot/dovecot.conf`.

```
!include_try /usr/share/dovecot/protocols.d/*.protocol
```

IMAPS and POP3S are more secure because they use SSL encryption to connect. A basic self-signed SSL certificate is automatically set up by package `ssl-cert` and used by Dovecot in `/etc/dovecot/conf.d/10-ssl.conf`.

`Mbox` format is configured by default, but you can also use `Maildir` if required. More details can be found in the comments in `/etc/dovecot/conf.d/10-mail.conf`. Also see [the Dovecot web site](https://doc.dovecot.org/admin_manual/mailbox_formats/) to learn about further benefits and details.

Make sure to also configure your chosen Mail Transport Agent (MTA) to transfer the incoming mail to the selected type of mailbox.

### [Restart the Dovecot daemon](https://ubuntu.com/server/docs/install-and-configure-dovecot#restart-the-dovecot-daemon)

Once you have configured Dovecot, restart its daemon in order to test your setup using the following command:

```
sudo service dovecot restart
```

Try to log in with the commands `telnet localhost pop3` (for POP3) or `telnet localhost imap2` (for IMAP). You should see something like the following:

```
bhuvan@rainbow:~$ telnet localhost pop3
Trying 127.0.0.1...
Connected to localhost.localdomain.
Escape character is '^]'.
+OK Dovecot ready.
```

## [Dovecot SSL configuration](https://ubuntu.com/server/docs/install-and-configure-dovecot#dovecot-ssl-configuration)

By default, Dovecot is configured to use SSL automatically using the package `ssl-cert` which provides a self signed certificate.

You can instead generate your own custom certificate for Dovecot using `openssh`, for example:

```
sudo openssl req -new -x509 -days 1000 -nodes -out "/etc/dovecot/dovecot.pem" \
    -keyout "/etc/dovecot/private/dovecot.pem"
```

Next, edit `/etc/dovecot/conf.d/10-ssl.conf` and amend following lines to specify that Dovecot should use these custom certificates :

```
ssl_cert = </etc/dovecot/private/dovecot.pem
ssl_key = </etc/dovecot/private/dovecot.key
```

You can get the SSL certificate from a Certificate Issuing Authority or you can create self-signed one. Once you create the certificate, you will have a key file and a certificate file that you want to make known in the config shown above.

> **Further reading**:  
> For more details on creating custom certificates, see our guide on [security certificates](https://ubuntu.com/server/docs/certificates).

## [Configure a firewall for an email server](https://ubuntu.com/server/docs/install-and-configure-dovecot#configure-a-firewall-for-an-email-server)

To access your mail server from another computer, you must configure your firewall to allow connections to the server on the necessary ports.

- IMAP - 143
- IMAPS - 993
- POP3 - 110
- POP3S - 995

## [References](https://ubuntu.com/server/docs/install-and-configure-dovecot#references)