# Bouler einrichten

`git clone https://github.com/letsencrypt/boulder/`
`cd boulder`

_Server starten_

`docker compose up`

_Local dns address finden_

`ifconfig docker0 | grep "inet addr:" | cut -d: -f2 | awk '{ print $1}'`

_IP setzen für DNS_
```sh
docker compose run --use-aliases -e FAKE_DNS=$(ifconfig docker0 | grep "inet addr:" | cut -d: -f2 | awk '{ print $1}') --service-ports boulder ./start.py
```

_Benutzung mit Certbot_

```sh
SERVER=http://localhost:4001/directory certbot_test certonly --standalone -d test.example.com
```

_Benutzen mit anderen ACME Clients_

```sh
ACME v2, HTTP: `http://localhost:4001/directory`
ACME v2, HTTPS: `https://localhost:4431/directory`
```