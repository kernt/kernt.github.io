---
tags:
  - deb822
  - repository
  - Debian
---
# [FORMAT IM DEB822-STIL](https://manpages.debian.org/unstable/apt/sources.list.5.de.html#FORMAT_IM_DEB822-STIL)

Dateien in diesem Format haben die Endung *.sources* . Dieses Format hat eine ähnliche Syntax wie andere von Debian und seinen Derivaten benutzte Dateien, wie Metadatendateien, die APT von den konfigurierten Quellen herunterlädt oder der Datei debian/control in einem Debian-Quellpaket. Individuelle Einträge werden durch eine leere Zeile getrennt; zusätzliche leere Zeilen werden ignoriert und #-Zeichen am Anfang einer Zeile kennzeichnen die ganze Zeile als Kommentar.
Ein Eintrag kann daher deaktiviert werden, indem jede Zeile, die zum Absatz gehört, auskommentiert wird. Es ist üblicherweise jedoch einfacher, dem Absatz das Feld »Enabled: no« hinzuzufügen, um den Eintrag zu deaktivieren. Durch Entfernen des Feldes oder indem es auf »yes« gesetzt wird, wird es wieder aktiviert. Optionen haben dieselbe Syntax wie jedes andere Feld: ein Feldname, durch einen Doppelpunkt (:) und optionale Leerräume von ihren (ihrem) Wert(en) getrennt. Beachten Sie insbesondere, dass mehrere Werte durch Leerräume (wie Leerzeichen, Tabulatoren und Zeilenumbrüche) getrennt werden, nicht durch Kommas, wie im einzeiligen Format. Felder mit mehreren Werten wie Architectures haben obendrein Architectures-Add und Architectures-Remove, um den Vorgabewert zu ändern, statt ihn zu ersetzen.

Dies ist ein neues Format, das von APT selbst seit Version 1.1. unterstützt wird. Ältere Versionen ignorieren solche Dateien, wie vorher beschrieben, mit einer Benachrichtigung. Es ist vorgesehen, dieses Format schrittweise zum Standardformat zu machen und das vorher beschriebene Format mit dem einzeiligen Stil zu missbilligen, da das neue für Menschen und Maschinen gleichermaßen einfacher zu erstellen, zu erweitern und zu ändern ist, insbesondere dann, wenn viele Quellen und/oder Optionen beteiligt sind. Entwickler, die mit APT-Quellen arbeiten und/oder sie auswerten, sind dringend ermutigt, dieses Format zu unterstützen und das APT-Team zu kontaktieren, um diese Arbeit zu koordinieren und weiterzugeben. Benutzer können dieses Format bereits übernehmen, es könnten jedoch Probleme mit Software auftreten, die dieses Format noch nicht unterstützen.
# Debian vereinfacht Umstellung auf deb822

**Quelleneinträge im alten Format ausgeben**

`apt modernize-sources`

Simulationsmodus, den man per N

Quelle: https://linuxnews.de/debian-vereinfacht-umstellung-auf-deb822/

https://docs.ansible.com/ansible/latest/collections/ansible/builtin/deb822_repository_module.html#ansible-builtin-deb822-repository-module-add-and-remove-deb822-formatted-repositories
```
---
- name: Add debian repo
  deb822_repository:
    name: debian
    types: deb
    uris: http://deb.debian.org/debian
    suites: stretch
    components:
      - main
      - contrib
      - non-free

- name: Add debian repo with key
  deb822_repository:
    name: debian
    types: deb
    uris: https://deb.debian.org
    suites: stable
    components:
      - main
      - contrib
      - non-free
    signed_by: |-
      -----BEGIN PGP PUBLIC KEY BLOCK-----

      mDMEYCQjIxYJKwYBBAHaRw8BAQdAD/P5Nvvnvk66SxBBHDbhRml9ORg1WV5CvzKY
      CuMfoIS0BmFiY2RlZoiQBBMWCgA4FiEErCIG1VhKWMWo2yfAREZd5NfO31cFAmAk
      IyMCGyMFCwkIBwMFFQoJCAsFFgIDAQACHgECF4AACgkQREZd5NfO31fbOwD6ArzS
      dM0Dkd5h2Ujy1b6KcAaVW9FOa5UNfJ9FFBtjLQEBAJ7UyWD3dZzhvlaAwunsk7DG
      3bHcln8DMpIJVXht78sL
      =IE0r
      -----END PGP PUBLIC KEY BLOCK-----

- name: Add repo using key from URL
  deb822_repository:
    name: example
    types: deb
    uris: https://download.example.com/linux/ubuntu
    suites: '{{ ansible_distribution_release }}'
    components: stable
    architectures: amd64
    signed_by: https://download.example.com/linux/ubuntu/gpg
```

Neu in Ansible Core 2.15

https://docs.ansible.com/ansible/latest/roadmap/ansible_core_roadmap_index.html