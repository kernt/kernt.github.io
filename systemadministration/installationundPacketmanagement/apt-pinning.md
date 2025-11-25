---
tags:
  - apt
  - packet-management
  - system-administration
  - pinning
---
# Warum Apt-Pinning?

Bei Debian hat man oft nur recht alte Pakete zur Verfügung (der Schwerpunkt liegt auf Stabilität und nicht auf Aktualität).
Manchmal braucht man aber aktuellere Pakete. Hier wird erklärt, wie man die Pakete mischen kann, um auch unter Sarge die Testing und Sid/Unstable Pakete nutzen zu können.

1. Warum sollte man so etwas tun?

   Im Gegensatz zur Stable-Distribution werden Testing und Unstable nicht vom Security-Team betreut. Bei vielen Paketen, z.B. X, ist das allerdings kein großes Problem. Außerdem haben die neueren Versionen meist auch zusätzliche Features und weniger Bugs.

2. sources.list

   Als erstes passen wir die /etc/apt/sources.list an und setzen alle Sourcen ein. Ein Beispiel, wie sie nun aussehen könnte: (siehe auch Debian/sources.list)

/etc/apt/sources.list

```sh
deb http://ftp3.de.debian.org/debian/ squeeze main non-free contrib
deb-src http://ftp3.de.debian.org/debian/ squeeze main non-free contrib

deb http://ftp2.de.debian.org/debian sid main contrib non-free
deb-src ftp://ftp2.de.debian.org/debian sid main contrib non-free

deb http://ftp2.de.debian.org/debian testing main contrib non-free
deb-src http://ftp2.de.debian.org/debian testing main contrib non-free

deb http://ftp2.de.debian.org/debian stable main contrib non-free
deb-src http://ftp2.de.debian.org/debian stable main contrib non-free

deb http://security.debian.org/ stable/updates main contrib non-free
deb http://security.debian.org/ testing/updates main contrib non-free
```
### preferences

Als nächstes erstellen wir die Datei /etc/apt/preferences, sofern sie noch nicht vorhanden ist. Hier kommen nun die eigentlichen Einträge für das Apt-Pinning hinein.
Normalerweise haben die Pakete die höchste Priorität, die die neueste Versions-Nummer besitzen. Durch unseren jetzigen Eintrag wird dies aber verändert: apt/PinPriorität

```sh
Package: *
Pin: release v=3.1*
Pin-Priority: 700

Package: *
Pin: release a=testing
Pin-Priority: 90

Package: *
Pin: release a=unstable
Pin-Priority: 50
```

Siehe: http://www.de.debian.org/doc/manuals/debian-reference/ch-package.de.html#s-apt-tracking

Die höchste Priorität (Wert) gibt an, welche Version von apt installiert würde.
Die Namen wie a=stable kommen aus dem Release file im Debian Repository. siehe auch dpkg/LokaleDebDateien

Auszug aus  apt-cache policy

```sh
500 http://ftp2.de.debian.org stable/contrib Packages
     release v=3.0r2,o=Debian,a=stable,l=Debian,c=contrib
     origin ftp2.de.debian.org

Oder ein eigenes Repository durch pinning auf 900 gesetzt:

900 http://lap.local.invalid woody/contrib Packages
     release v=3.0r2,o=Debian,a=stable,l=Debian,c=contrib
     origin lap.local.invalid
```

5. apt-get update

Als nächstes lassen wir ein einfaches apt-get update laufen. Während des Updates bekommen wir wahrscheinlich folgende Fehlermeldung:

```
E: Dynamic MMap ran out of room
E: Error occured while processing sqlrelay-sqlite (NewPackage)
E: Problem with MergeList /var/lib/apt/lists/ftp.us.debian.org_debian_dists_woody_contrib_binary-i386_Packages
E: The package lists or status file could not be parsed or opened.
```

Dies passiert, da der Cache von apt nicht groß genug ist, um alle pakete von stable, testing und unstable zu verarbeiten. Um dies zu umgehen, fügen wir folgende Zeile in die Datei /etc/apt/apt.conf ein.

`APT::Cache-Limit "8388608"`
## Pakete installieren

Wenn wir nun ein normales apt-get install _paket_ ausführen, würde es die Stable oder die Version installieren, die die höchste Priorität besitzt (/etc/apt/preferences). Um nun Pakete einer anderen Version zu installieren, haben wir mehrere möglichkeiten.

Um ein Paket aus Unstable zu installieren und zu versuchen die Abhängigkeiten aus Stable zu beziehen installieren wir wie folgt:

`apt-get install <paket>/unstable`
## apt-get install zsh/unstable

```sh
    Reading Package Lists... Done
    Building Dependency Tree... Done
    Selected version 4.0.6-7 (Debian:unstable) for zsh
    Some packages could not be installed. This may mean that you have
    requested an impossible situation or if you are using the unstable
    distribution that some required packages have not yet been created
    or been moved out of Incoming.
    Since you only requested a single operation it is extremely likely that
    the package is simply not installable and a bug report against
    that package should be filed.
    The following information may help to resolve the situation:
    Sorry, but the following packages have unmet dependencies:
    zsh: Depends: libc6 (>= 2.2.5-13) but 2.2.5-11.1 is to be installed
    E: Sorry, broken packages
    
    Um dem zu entgehen, installieren wir wie folgt:
    
    apt-get install -t unstable <paket>
    
    Dies wird alle nötigen Abhängigkeiten aus dem Unstable beziehen wie im Beispiel zu sehen:
    
    # apt-get -t unstable install zsh
    Reading Package Lists... Done
    Building Dependency Tree... Done
    
    The following extra packages will be installed:
    libc6 libc6-dev libc6-pic libdb1-compat locales
    The following NEW packages will be installed:
    libdb1-compat
    
    5 packages upgraded, 1 newly installed, 0 to remove and 394 not upgraded.
    Need to get 11.6MB of archives. After unpacking 606kB will be used.
    Do you want to continue? [Y/n]
```

**Quelle:**

[Apt Pinning](http://linuxwiki.de/Debian/AptPinning)