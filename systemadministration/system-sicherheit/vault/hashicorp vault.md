---
tags:
  - vault
  - sicherheit
---
# Vault  Servereinrichten

Daten werden in `/opt/vault/data` wie es dann in der `VAULT_DATA` environment variable wieder zu finden ist.

[environment variablen](https://developer.hashicorp.com/vagrant/docs/other/environmental-variables):

- VAULT_DATA
- VAULT_CONFIG

> Die UI wird ein auffordern einen gpg Key zu benutzen. Um das zu verhindern bietet sich die Konsole an, aber damit das gehet mus temporär ui = false gesetzt werden!

# Vault Benutzer Berechtigungen einrichten

Um einen System user _vault_ einzurichten wird das Vault Data Verzeichnis als HomeDir gesetzt und eine nologin shell.

`sudo useradd --system --home ${VAULT_DATA} --shell /sbin/nologin vault`

> Bei eine Packet basierenden installation sind passende Berechtigunen schon gesetzt

# Vault zurück setzen

Besonders bei der Ersten Einrichtung kann es vorkommen das man nochmal anfangen muss da man eine Fehler hat aus unbekanten Gründen dann wird oft aber schon alles angelegt was man braucht nur hat mal dann die infos um die Vault zu öffenen nicht etc.

## Vault reseten wenn das beckend file ist

Dazu muss einfach das Storage Verzeichnis vollstängig gelöscht werden

`rm -rf /opt/vault/data*`

##  Vault reseten wenn das beckend eine Postgresql Datenbank ist

```sql
psql -U myvaultdbuser -h myvaultDB.host.name -p5432 vaultdatabasname -c 'truncate table vault_kv_store';
```

##  Vault reseten wenn das beckend eine 

> Etcd < v3.5 ist deprecated

##  Vault reseten wenn das beckend eine MySQL Datenbank ist


# Vault Secret Engines einrichten

Secrets die von der vault unterstüzt werden.

## Vault Secret Engine Consul

## Vault Secret Engine Kubernetes

https://developer.hashicorp.com/vault/docs/secrets/kubernetes

## Vault Secret Engine PKI

# Vault Auth Methods

Methoden zur Anmeldung

## Vault Auth Method Kubernetes

https://developer.hashicorp.com/vault/docs/auth/kubernetes

## Vault Auth Method MFA

https://developer.hashicorp.com/vault/docs/auth/login-mfa

## Vault Auth GitHub

https://developer.hashicorp.com/vault/docs/auth/github

