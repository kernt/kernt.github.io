# ubuntu Pro

Pro Subcription

Der Zugang zu den Ubuntu Pro Repositories ist per Bearer Token/Basic HTTP Authentication guarded.

https://cstan.io/en/post/2024/04/ubuntu-pro-mirror/

Token generieren
In docker Maschine einloggen, z.B. fravm007131

ubuntu image laden

`docker pull ubuntu:22.04`

ubuntu container starten

`docker run --rm -it --entrypoint bash ubuntu:22.04`

```sh
apt update
apt install ubuntu-advantage-tools ca-certificates -y
```

Attach machine:
via token
copy attach command from Ubuntu Pro Dashboard in Subscription tab
root@ab366220eaec:/# pro attach C12KdLCk...
Enabling Ubuntu Pro: ESM Infra
Ubuntu Pro: ESM Infra enabled
This machine is now attached to 'Ubuntu Pro (Infra-only) + Support (24/7)'
 
```sh
SERVICE          ENTITLED  STATUS       DESCRIPTION
esm-infra        yes       enabled      Expanded Security Maintenance for Infrastructure
fips-preview     yes       disabled     Preview of FIPS crypto packages undergoing certification with NIST
fips-updates     yes       disabled     FIPS compliant crypto packages with stable security updates
usg              yes       disabled     Security compliance and audit tools
```
 
NOTICES
Operation in progress: pro attach
 
```
For a list of all Ubuntu Pro services, run 'pro status --all'
Enable services with: pro enable _service_
 
                Account: Operational services GmbH & Co. KG
           Subscription: Ubuntu Pro (Infra-only) + Support (24/7)
            Valid until: Tue Jul 30 23:59:59 2024 UTC
Technical support level: advanced
```

Mindestens esm-infra  muss enabled  sein,
App Updates wurden nicht eingekauft
token kopieren

```
root@ab366220eaec:/# cat /etc/apt/auth.conf.d/90ubuntu-advantage
machine esm.ubuntu.com/infra/ubuntu/ login bearer password mAgJSEWNBSWtwak9IcHp1RH....# ubuntu-pro-client
```

## GPG Key beziehen

Genauso wir bei der Token-generierung vorgehen und Maschine/Container registrieren

In order for Uyuni to synchronize the repositories, the corresponding GPG keys (ubuntu-pro-esm-infra.gpg, ubuntu-pro-esm-apps.gpg) must be imported. These are located on a system registered with Ubuntu Pro under /etc/apt/trusted.gpg.d and can be imported as follows:

`gpg --homedir /var/lib/spacewalk/gpgdir --import ubuntu-pro-*.gpg`

In einem registrierten System die Datei /etc/apt/trusted.gpg.d/ubuntu-pro-esm-infra.gpg extrahieren
z.B. im laufenden Container per base64 encoden

```
root@ab366220eaec:~ # base64 /etc/apt/trusted.gpg.dubuntu-pro-esm-infra.gpg
mQINBFy2kH0BEADl/2e2pULZaSRovd3E1i1cVk3zebzndHZm/hK8/Srx69ivw3pY680gFE/N3s3R
/C5Jh9ThdD1zpGmxVdqcABSPmW1FczdFZY2E37HMH7Uijs4CsnFs8nrNGQaqX/T1g2fQqjia3zka
bMeehUEZC5GPYjpeeFW6Wy1O1A1Tzu7/Wjc+uF/tYYe/ZPXea74QZphu/N+8dy/ts/IzL2VtXuxi
egGLfBFqzgZuBmlxXHVhftKvcis9t2ko65uVyDcLtItMhSJokKBsIYJliqOXjUbQf5dz8vLXkku9
4arBMgsxDWT4K/xIOTsaI/GMlSIKQ6Ucd/GKrBEsy5O8RDtD9A2klV7YeEwPEgqL+RhpdxAs/xUe
TOZGJKwuvlBjzIhJF9bIfbyzx7DdcGFqRE+a8eBIUMQjVkt9Yk7jj0eV3oVTE7XNhb53rHuPL+zJ
....
```
gpg file auf den SUMA/Uyuni kopieren und in den Keyring importieren

gpg --homedir /var/lib/spacewalk/gpgdir --import ubuntu-pro-*.gpg
Repositories und Channels anlegen nach diesem Schema:

```
https://bearer:<token>@esm.ubuntu.com/infra/ubuntu/dists/<release>-infra-updates/main/binary-<arch>/	Operating system functional updates
https://bearer:<token>@esm.ubuntu.com/infra/ubuntu/dists/<release>-infra-security/main/binary-<arch>/	Operating system security updates
https://bearer:<token>@esm.ubuntu.com/apps/ubuntu/dists/<release>-apps-updates/main/binary-<arch>/	Application functional updates
https://bearer:<token>@esm.ubuntu.com/apps/ubuntu/dists/<release>-apps-security/main/binary-<arch>/	Application security updates

```

arch and release must each be replaced with one of the following values:

amd64, arm64, armel, armhf, i386, powerpc, ppc64el, s390x	bionic, focal, jammy, noble, trusty, xenial

The steps should also be applicable for the additional repositories fips, fips-updates, cis, realtime and ros