# [MinIO Client](https://min.io/docs/minio/linux/reference/minio-mc.html#minio-client "Permalink to this heading")

The MinIO Client [`mc`](https://min.io/docs/minio/linux/reference/minio-mc.html#command-mc "mc") command line tool provides a modern alternative to UNIX commands like `ls`, `cat`, `cp`, `mirror`, and `diff` with support for both filesystems and Amazon S3-compatible cloud storage services.

The **mc** commandline tool is built for compatibility with the AWS S3 API and is tested with MinIO and AWS S3 for expected functionality and behavior.

See [Command Quick Reference](https://min.io/docs/minio/linux/reference/minio-mc.html#minio-mc-commands) for a list of supported commands.
## Minio Admin Cli installieren

[optional-minio-admin-cli-installieren](https://teqqy.de/s3-speicher-mit-minio-und-nginx-tutorial-ohne-docker/#optional-minio-admin-cli-installieren)

Gegebenenfalls brauchst du zu einem späteren Zeitpunkt mal die Admin CLI von Minio. Also solltest du diese direkt mit installieren. Diese ist ein einfaches binary das wir nur auf das System kopieren müssen.

```bash
curl https://dl.min.io/client/mc/release/linux-amd64/mc -o /usr/local/sbin/mc
```

Allerdings wird es ohne Execute Rechte nicht ausgeführt werden können, also fügen wir diese noch hinzu

```bash
chmod +x /usr/local/sbin/mc
```

Anschließend könntest du mittels **mc** den Minio Server auch via CLI administrieren.

Use the [`mc alias set`](https://min.io/docs/minio/linux/reference/minio-mc/mc-alias-set.html#command-mc.alias.set "mc.alias.set") command to add an Amazon S3-compatible service to the [`mc`](https://min.io/docs/minio/linux/reference/minio-mc.html#command-mc "mc") [configuration](https://min.io/docs/minio/linux/reference/minio-mc.html#mc-configuration).

```sh
bash +o history
mc alias set ALIAS HOSTNAME ACCESS_KEY SECRET_KEY
bash -o history
```
# mc alias

**Aliases auflisten**

`mc alias list`

```sh
root@srv3:~# mc alias list
gcs  
  URL       : https://storage.googleapis.com
  AccessKey : YOUR-ACCESS-KEY-HERE
  SecretKey : YOUR-SECRET-KEY-HERE
  API       : S3v2
  Path      : dns
  Src       : /root/.mc/config.json

local
  URL       : http://localhost:9000
  AccessKey : 
  SecretKey : 
  API       : 
  Path      : auto
  Src       : /root/.mc/config.json

play 
  URL       : https://play.min.io
  AccessKey : Q3AM3UQ867SPQQA43P2F
  SecretKey : zuf+tfteSlswRu7BJ86wekitnifILbZam1KYY3TG
  API       : S3v4
  Path      : auto
  Src       : /root/.mc/config.json

s3   
  URL       : https://s3.amazonaws.com
  AccessKey : YOUR-ACCESS-KEY-HERE
  SecretKey : YOUR-SECRET-KEY-HERE
  API       : S3v4
  Path      : dns
  Src       : /root/.mc/config.json

```

**Alias anlegen**

`mc alias set myminio https://minio.tuxcoach.de minioadmin 'PASSWORD'`
# mc admin

**Test the Connection**

`mc admin info myminio`

**Updates einspielen**

`mc admin update myminio`

https://min.io/docs/minio/linux/reference/minio-mc.html

