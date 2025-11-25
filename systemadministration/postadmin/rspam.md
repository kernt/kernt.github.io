# [OpenBSD - OpenSMTPD](https://blog.stoege.net/posts/opensmtpd/)

## Running a Mailserver on OpenBSD …

### Source

- [https://karchnu.fr/posts/2020-09-17-certificate-smtp-imap-antispam.html](https://karchnu.fr/posts/2020-09-17-certificate-smtp-imap-antispam.html)
### Requirements

- OpenBSD VM
- Public IP & FQDN
- no Portfilter from Hoster
- root permission

### Packages

```.sh
pkg_add opensmtpd-extras opensmtpd-filter-rspamd dovecot dovecot-pigeonhole redis rspamd-- opensmtpd-filter-senderscore
```
### FQDN

```sh
export host="hostname"
export domain="domain.tld"
export fqdn="${host}.${domain}"
```
### httpd.conf

```sh
f="/etc/httpd.conf"; test -f ${f} && cp ${f} "${f}-$(date +'%s')"

cat << EOF > ${f}
# added $(date)
server "${fqdn}" {
  listen on * port 80
  location "/.well-known/acme-challenge/*" {
    root "/acme"
    request strip 2
  }
}
EOF
chown root:wheel ${f}; chmod 644 ${f}
```
### pf.conf

allow Certain Ports for Any

https://karchnu.fr/posts/2020-09-17-certificate-smtp-imap-antispam.html