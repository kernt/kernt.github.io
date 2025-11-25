---
tags:
  - sicherheit
  - certutil
---
# Certutil

[Certutil](certutil.md)- Manage keys and certificate in both NSS databases and other NSS tokens

## Installation von [Certutil](certutil.md)

```sh
    Debian/Ubuntu: sudo apt-get install libnss3-tools
    Fedora: su -c "yum install nss-tools"
    Gentoo: su -c "echo 'dev-libs/nss utils' >> /etc/portage/package.use && emerge dev-libs/nss" (You need to launch all commands below with the nss prefix, e.g., nsscertutil.)
    Opensuse: sudo zypper install mozilla-nss-tools
```

Verzeichnisse erstellen :

```sh
sudo mkdir -p /etc/pki/tls/certs
sudo mkdir /etc/pki/tls/private
```

**Quellen:**

* [redhat certutil](https://access.redhat.com/documentation/en-US/Red_Hat_Directory_Server/8.1/html/Administration_Guide/Managing_SSL-Using_certutil.html)
* [Man Page Ubuntu](http://manpages.ubuntu.com/manpages/zesty/man1/certutil.1.html)
* [adding-a-certificate-authority-to-the-trusted-list-in-ubuntu](http://blog.tkassembled.com/410/adding-a-certificate-authority-to-the-trusted-list-in-ubuntu/)
* [linux_cert_management](https://chromium.googlesource.com/chromium/src/+/master/docs/linux_cert_management.md)
