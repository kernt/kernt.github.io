### Let’s encrypt Zertifikat mit Certbot beantragen

[#](https://teqqy.de/s3-speicher-mit-minio-und-nginx-tutorial-ohne-docker/#lets-encrypt-zertifikat-mit-certbot-beantragen)

Zuerst richten wir noch die Punkte für die DNS Challenge zu Cloudflare ein. Dazu erstellen wir uns ein Verzeichnis

```bash
mkdir -p /etc/certbot
```

und legen uns dort eine Datei an

```bash
touch /etc/certbot/credentials
```

in diese kommt nun unser Cloudflare Token. Also editieren wir die Datei

```bash
vim /etc/certbot/credentials
```

und kopieren folgendes dort rein

```
dns_cloudflare_api_token={DEIN_CLOUDFLARE_API_TOKEN}
```

Damit die Datei nicht von jedem Nutzer auf dem System gelesen werden kann, passen wir noch die Berechtigungen mittels

```bash
chmod 600 /etc/certbot/credentials
```

an. Nun haben wir den Grundstock gelegt um das Zertifikat beantragen zu können. Beachte jetzt, dass du die Domain entsprechend an deine Domain anpassen musst.

```bash
certbot certonly --dns-cloudflare --dns-cloudflare-credentials /etc/certbot/credentials -d minio.example.com -d s3.example.com
```

Aus irgendwelchen Gründen funktioniert das bei mir beim ersten mal nicht immer zuverlässig. Solltest du auf den Fehler stoßen, dass man überprüfen soll, ob der DNS Eintrag gesetzt ist, warte noch kurz und führe den Befehl erneut aus.