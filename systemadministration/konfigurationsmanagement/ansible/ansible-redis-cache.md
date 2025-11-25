---
tags:
  - konfigurationsmanagement
  - ansible
  - cache
  - redis
  - config-db
---
# Ansible redis cache facts einrichten

**Redis Installation**

`sudo dnf install -y redis python3-redis`

**Redis Aktivieren**

`systemctl enable --now redis`

**Ansible Zeitmessung ohne Redis**

`time ansible localhost -m setup`

**Redis Environment var setzen**

`export ANSIBLE_CACHE_PLUGIN=redis`

**Anpassungen an der ansible.cfg**

```ini
[defaults]
fact_caching=redis
fact_caching_timeout = 7200
fact_caching_connection = localhost:6379:0
```

## Die Konfigurationswerte stehen für Folgendes

fact_caching gibt an, welches Cache-Plugin verwendet wird.
fact_caching_timeout legt die Verfallszeit in Sekunden für die Daten des Cache-Plugins fest. Um sicherzustellen, dass die Daten nicht ablaufen, setzen Sie diese Option auf 0.
Es ist jedoch wichtig, diese Option in der Produktion auf einen vernünftigen Wert zu setzen, je nachdem, wie sich Ihre Daten ändern.
Standardmäßig ist dieser Wert auf 86400s eingestellt, was 24 Stunden entspricht. Sie können einen niedrigeren Wert einstellen und den Prozess automatisieren, um die Daten automatisch zu aktualisieren. In diesem Beispiel habe ich den Wert auf zwei Stunden gesetzt.
Wenn fact_caching_timeout abläuft, funktionieren alle Playbooks, die Fakten benötigen und bei denen die Option gather_facts auf false gesetzt ist, nicht mehr. Damit sie funktionieren, aktualisieren Sie die Faktdaten im Cache.
fact_caching_connection ist eine durch Doppelpunkte getrennte Zeichenfolge, die Redis-Verbindungsinformationen darstellt.

Dss Format ist `host:port:db:password`. 
Zum Beispel:

`localhost:6379:0:password`

## Testen des Fact Caches

**Run mit dem Setup Ansible Module**

`ansible localhost -m setup`

Jetzt sollte ein Playbook auch mit`gather_facts: false`  facts liefern

**Test der facts aus redis**

`time ansible-playbook ansible_facts.yml`

source [ansible-fact-cache-redis](https://www.redhat.com/sysadmin/ansible-fact-cache-redis)
