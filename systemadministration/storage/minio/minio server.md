---
tags:
  - minio
  - storage
---
# minio allegemein

MinIO is a software-defined high performance distributed object storage server. You can run MinIO on consumer or enterprise-grade hardware and a variety of operating systems and architectures.

All MinIO deployments implement [Erasure Coding](https://min.io/docs/minio/linux/operations/concepts/erasure-coding.html#minio-erasure-coding) backends. You can deploy MinIO using one of the following topologies:

[Single-Node Single-Drive](https://min.io/docs/minio/linux/operations/install-deploy-manage/deploy-minio-single-node-single-drive.html#minio-snsd) (SNSD or “Standalone”)

Local development and evaluation with no/limited reliability

[Single-Node Multi-Drive](https://min.io/docs/minio/linux/operations/install-deploy-manage/deploy-minio-single-node-multi-drive.html#minio-snmd) (SNMD or “Standalone Multi-Drive”)

Workloads with lower performance, scale, and capacity requirements
Drive-level reliability with configurable tolerance for loss of up to 1/2 all drives
Evaluation of multi-drive topologies and failover behavior.

[Multi-Node Multi-Drive](https://min.io/docs/minio/linux/operations/install-deploy-manage/deploy-minio-multi-node-multi-drive.html#minio-mnmd) (MNMD or “Distributed”)

Enterprise-grade high-performance object storage

Multi Node/Drive level reliability with configurable tolerance for loss of up to 1/2 all nodes/drives
Primary storage for AI/ML, Distributed Query, Analytics, and other Data Lake components
Scalable for Petabyte+ workloads - both storage capacity and performance
# minio Installation

**Download und installation**

```sh
wget https://dl.min.io/server/minio/release/linux-amd64/minio
sudo mv minio /usr/local/bin/
sudo chmod +x /usr/local/bin/minio
chown minio:minio /usr/local/bin/minio
```

**minio user anlegen**

```sh
sudo useradd -r minio-user -s /sbin/nologin
```

**Verzeichnisse für minio vorbereiten**

```sh
sudo mkdir /home/minio-user
sudo chown minio-user:minio-user /home/minio-user
```

**systemd service für minio anlegen**

_/etc/systemd/system/minio.service_

```
cat << EOF > /etc/systemd/system/minio.service
[Unit]
Description=MinIO
Documentation=https://docs.min.io
Wants=network-online.target
After=network-online.target

[Service]
User=minio-user
Group=minio-user
EnvironmentFile=-/etc/default/minio
ExecStart=/usr/local/bin/minio server /data
Restart=always
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target

EOF
```

oder von minio Kopieren

`wget https://raw.githubusercontent.com/minio/minio-service/master/linux-systemd/minio.service`

**Konfiguration der environment variablen**

*/etc/default/minio*

```sh
IP=$(hostname -i | cut -d " " -f 2)
cat << EOF > /etc/default/minio
# minio Server
MINIO_ACCESS_KEY="minioadmin"
MINIO_VOLUMES="/mnt/sam500tb"
MINIO_OPTS="-C /etc/minio --address 0.0.0.0:9400 --console-address ${ip}:9401"
MINIO_ROOT_PASSWORD="$PASSWORD"
MINIO_ROOT_USER="minioadmin"
# minio Server SFTP options https://min.io/docs/minio/linux/developers/file-transfer-protocol.html
#
# add user ssh-keygen -s ~/.ssh/ca_user_key -I miniouser -n miniouser -V +1h -z 1 miniouser1.pub
#
# --sftp="address=:8022"
# --sftp="ssh-private-key=/home/miniouser/.ssh/id_rsa"
#
# minio Logger https://min.io/docs/minio/windows/operations/monitoring/minio-logging.html?ref=con
#
#MINIO_LOGGER_WEBHOOK_ENABLE_<IDENTIFIER>="on"
#MINIO_LOGGER_WEBHOOK_ENDPOINT_<IDENTIFIER>="https://minio.tuxcoach.de"
#MINIO_LOGGER_WEBHOOK_AUTH_TOKEN_<IDENTIFIER>="TOKEN"

EOF
```

**Starten und aktiviren des minio Services**

```shell
sudo systemctl daemon-reload
sudo systemctl start minio
sudo systemctl enable minio
```
# minio Konfiguration

**Konfigurationsverzeichnis anlegen**

*/mnt/minio*

```sh
chown minio:minio /mnt/minio
```

**Default minio Konfiguration anpassen**

```
MINIO_ACCESS_KEY="minioaccesskey"
MINIO_VOLUMES="/mnt/minio"
MINIO_OPTS="-C /etc/minio --address 0.0.0.0:9000"
MINIO_SECRET_KEY="miniosecretkey"
```

## minio mit TLS absichern

Unter  */home/minio-user/.minio/certs/* müssen die zertifikate ab

`/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES`

**set permission**

```sh
chown minio-user:minio-user /etc/minio/certs/private.key
chown minio-user:minio-user /etc/minio/certs/public.crt
```
# minio Administration

**Server Starten**

```sh
/usr/local/bin/minio server --address :9000 --console-address :9099 --certs-dir /home/minio/.minio/certs/ /mnt/data1/minio /mnt/data2/minio
```

`mini mc` 
`mc update`

```sh
root@srv1:~# mc alias ls
gcs    
  URL       : https://storage.googleapis.com
  AccessKey : YOUR-ACCESS-KEY-HERE
  SecretKey : YOUR-SECRET-KEY-HERE
  API       : S3v2
  Path      : dns

local  
  URL       : http://localhost:9000
  AccessKey : 
  SecretKey : 
  API       : 
  Path      : auto

myminio
  URL       : http://192.168.4.41:9000
  AccessKey : minioadmin
  SecretKey : minioadmin
  API       : s3v4
  Path      : auto

play   
  URL       : https://play.min.io
  AccessKey : Q3AM3UQ867SPQQA43P2F
  SecretKey : zuf+tfteSlswRu7BJ86wekitnifILbZam1KYY3TG
  API       : S3v4
  Path      : auto

s3     
  URL       : https://s3.amazonaws.com
  AccessKey : YOUR-ACCESS-KEY-HERE
  SecretKey : YOUR-SECRET-KEY-HERE
  API       : S3v4
  Path      : dns
```

`mc alias set myminio https://192.168.4.41:9000 minioadmin minioadmin`

`mc alias set myminio https://srv1.tuxsupport.de:9000 minioadmin minioadmin`

```sh
mc admin 
    service              restart, stop and unfreeze a MinIO cluster  
    update               update all MinIO servers 
    info                 display MinIO server information
    user                 manage users
    group                manage groups
    policy               manage policies defined in the MinIO server        
    replicate            manage MinIO site replication
    idp                  manage MinIO IDentity Provider server configuration
    config               manage MinIO server configuration
    decommission, decom  manage MinIO server pool decommissioning
    heal                 heal bucket(s) and object(s) on MinIO server       
    prometheus           manages prometheus config
    kms                  perform KMS management operations

    /home/minio/.minio/certs/
```

Quellen:
# minio Fehlerbehebung

## Fehler bei der initialen Ausführung des Servers

Mit journalctl nach -u die Unit = minio eingrenzen und nach Datum mit --since.  --no-pager

`journalctl -u minio --since "2025-01-01 18:53" --no-pager`

```sh
Jan 01 18:53:36 srv3.local minio[532906]: Error: unable to rename (/mnt/sam500tb/.minio.sys/tmp -> /mnt/sam500tb/.minio.sys/tmp-old/53b7e145-bf74-4573-aa1a-5b4eaf8eb4e4) file access denied, drive may be faulty, please investigate (*fmt.wrapError)
Jan 01 18:53:36 srv3.local minio[532906]: Error: unable to rename (/mnt/sam500tb/.minio.sys/tmp -> /mnt/sam500tb/.minio.sys/tmp-old/53b7e145-bf74-4573-aa1a-5b4eaf8eb4e4) file access denied, drive may be faulty, please investigate (*fmt.wrapError)
Jan 01 18:53:36 srv3.local minio[532906]:        7: internal/logger/logger.go:268:logger.LogIf()
Jan 01 18:53:36 srv3.local minio[532906]:        6: cmd/logging.go:156:cmd.storageLogIf()
Jan 01 18:53:36 srv3.local minio[532906]:        5: cmd/prepare-storage.go:89:cmd.bgFormatErasureCleanupTmp()
Jan 01 18:53:36 srv3.local minio[532906]:        4: cmd/xl-storage.go:271:cmd.newXLStorage()
Jan 01 18:53:36 srv3.local minio[532906]:        3: cmd/object-api-common.go:63:cmd.newStorageAPI()
Jan 01 18:53:36 srv3.local minio[532906]:        2: cmd/format-erasure.go:571:cmd.initStorageDisksWithErrors.func1()
Jan 01 18:53:36 srv3.local minio[532906]:        1: github.com/minio/pkg/v3@v3.0.13/sync/errgroup/errgroup.go:123:errgroup.(*Group).Go.func1()
Jan 01 18:53:36 srv3.local minio[532906]: FATAL Unable to initialize backend: Unable to write to the backend
Jan 01 18:53:36 srv3.local minio[532906]:       > Please ensure MinIO binary has write permissions for the backend
Jan 01 18:53:36 srv3.local minio[532906]:       HINT:
Jan 01 18:53:36 srv3.local minio[532906]:         Run the following command to add write permissions: `sudo chown -R minio-user. <path> && sudo chmod u+rxw <path>`
```

Hier ist die Meldung *an 01 18:53:36 srv3.local minio[532906]: Error: unable to rename (/mnt/sam500tb/.minio.sys/tmp -> /mnt/sam500tb/.minio.sys/tmp-old/53b7e145-bf74-4573-aa1a-5b4eaf8eb4e4) file access denied, drive may be faulty, please investigate (*fmt.wrapError)* interesant  Obwohl zuvor Berechtigungen vergeben wurden, sind diese nicht bei der ersten Ausführung ausreichend gewesen.

Hier konnte dann mit nachfolgenden Kommando

`sudo chown -R minio-user. /mnt/sam500tb/ && sudo chmod u+rxw /mnt/sam500tb/`

das Problem erfolgreich behoben werden.
# minio server development

Note: You can access MinIO by using API.

In addition to the MinIO web console, you can use the API addresses to access the MinIO server and perform S3 operations. You can use different SDKs, such as boto3 and bulkboto3, to perform bucket and object operations to any Amazon S3-compatible object storage service. Applications can authenticate using the MINIO_ROOT_USER as aws_access_key_id and MINIO_ROOT_PASSWORD as aws_secret_access_key credentials. Please note that we need to communicate on port 9000 through MinIO APIs as we set in the docker-compose file. Here, provided a small Python code for creating and transferring a directory with its structure to the bucket using the bulkboto3 package; (7)

```PYTHON
import logging
 
from bulkboto3 import BulkBoto3
 
logging.basicConfig(
    level="INFO",
    format="%(asctime)s — %(levelname)s — %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)
logger = logging.getLogger(__name__)
 
TARGET_BUCKET = "test-bucket"
NUM_TRANSFER_THREADS = 50
TRANSFER_VERBOSITY = True
 
# instantiate a BulkBoto3 object
bulkboto_agent = BulkBoto3(
    resource_type="s3",
    endpoint_url="http://127.0.0.1:9000",
    aws_access_key_id="masoud",
    aws_secret_access_key="Strong#Pass#2022",
    max_pool_connections=300,
    verbose=TRANSFER_VERBOSITY,
)
 
# create a new bucket
bulkboto_agent.create_new_bucket(bucket_name=TARGET_BUCKET)
 
# upload a whole directory with its structure to an S3 bucket in multi thread mode
bulkboto_agent.upload_dir_to_storage(
    bucket_name=TARGET_BUCKET,
    local_dir="test_dir",
    storage_dir="my_storage_dir",
    n_threads=NUM_TRANSFER_THREADS,
)
```
# install server on GCE

`curl https://raw.githubusercontent.com/minio/docs/master/source/extra/examples/minio-dev.yaml -O`
# Installing MinIO to Kubernetes Using `Kubectl Krew` Command

Firstly, install "`krew`” . It is a plugin manager for `kubectl`. Run these commands to download and install `krew` as shown below. (10)

```
(  
  set -x; cd "$(mktemp -d)" &&  
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&  
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&  
  KREW="krew-${OS}_${ARCH}" &&  
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&  
  tar zxvf "${KREW}.tar.gz" &&  
  ./"${KREW}" install krew  
)
```

Add the `$HOME/.krew/bin` directory to your PATH environment variable. To do this, update your `.bashrc` file and append the following line, as shown below.

`export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"`

and restart your shell. Run `kubectl krew` to check the installation, as shown below.

Check the version of krew and its working using the command given below:

`kubectl krew version`
## Installing the MinIO Server

Now, we can install the MinIO Server using the following commands as shown in the images (11)

`kubectl krew install minio`

`kubectl minio init`

`kubectl minio tenant create tenant1 --servers 4 --volumes 16 --capacity 16Ti -n minio-operator`
# Installing MinIO To The Kubernetes Cluster Via Helm

We can also install MinIO to the Kubernetes cluster via Helm.

This chart bootstraps the MinIO Cluster on Kubernetes using the Helm package manager. (12)

Add repository:

`helm repo add minio-official https://charts.min.io`

Install chart:

`helm install my-minio minio-official/minio --version 5.0.15`

# Accessing The MinIO Console

Use the `**kubectl port-forward**` command to temporarily forward traffic from the MinIO pod to the local machine:

`kubectl port-forward pod/minio 9000 9090 -n minio-dev`

The command forwards the pod ports `**9000**` and `**9090**` to the matching port on the local machine while active in the shell. (**Note:** The `**kubectl port-forward**` command only functions while active in the shell session. Terminating the session closes the ports on the local machine.)

Access the [MinIO Console](https://min.io/docs/minio/kubernetes/gke/administration/minio-console.html#minio-console) by opening a browser on the local machine and navigating to `**http://**[**your-public-IP**](http://127.0.0.1:9001/)**:9001**`**.** **You can l**og in to the Console with the credentials `**minioadmin | minioadmin**`. These are the default [root user](https://min.io/docs/minio/kubernetes/gke/administration/identity-access-management/minio-user-management.html#minio-users-root) credentials.
# **Installing The** MinIO Object Storage For Linux

This procedure deploys a [Standalone](https://min.io/docs/minio/linux/operations/installation.html#minio-installation-comparison) MinIO server onto Linux for early development and evaluation of MinIO Object Storage and its S3-compatible API layer. (13) As with Linux, downloads are also available for [Mac](https://min.io/docs/minio/macos/index.html) and [Windows](https://min.io/docs/minio/windows/index.html).

Use the following commands to download the latest stable MinIO DEB (Debian/Ubuntu) and install it: (13)

`wget https://dl.min.io/server/minio/release/linux-amd64/archive/minio_20240214213602.0.0_amd64.deb -O minio.deb`  
`sudo dpkg -i minio.deb`
# References

(1) [https://github.com/minio/minio](https://github.com/minio/minio)  
(2) [https://github.com/minio/](https://github.com/minio/)  
(3) [https://medium.com/@rivera5656/unleashing-the-power-of-minio-your-gateway-to-scalable-object-storage-538fd18a059d](https://medium.com/@rivera5656/unleashing-the-power-of-minio-your-gateway-to-scalable-object-storage-538fd18a059d)  
(4) [https://argo-cd.readthedocs.io/en/stable/](https://argo-cd.readthedocs.io/en/stable/)  
(5) [https://www.sefidian.com/2022/04/08/deploy-standalone-minio-using-docker-compose/](https://www.sefidian.com/2022/04/08/deploy-standalone-minio-using-docker-compose/)  
(6)[https://github.com/minio/minio/blob/master/docs/orchestration/docker-compose/README.md](https://github.com/minio/minio/blob/master/docs/orchestration/docker-compose/README.md)  
(6.a)[https://github.com/minio/minio/blob/master/docs/orchestration/docker-compose/docker-compose.yaml](https://github.com/minio/minio/blob/master/docs/orchestration/docker-compose/docker-compose.yaml)  
(7) [https://www.sefidian.com/2022/04/08/deploy-standalone-minio-using-docker-compose/](https://www.sefidian.com/2022/04/08/deploy-standalone-minio-using-docker-compose/)  
(8) [https://min.io/docs/minio/kubernetes/upstream/index.html](https://min.io/docs/minio/kubernetes/upstream/index.html)  
(9) [https://min.io/docs/minio/kubernetes/gke/index.html](https://min.io/docs/minio/kubernetes/gke/index.html)  
(10) [https://krew.sigs.k8s.io/docs/user-guide/setup/install/](https://krew.sigs.k8s.io/docs/user-guide/setup/install/)  
(11) [https://min.io/download#/kubernetes](https://min.io/download#/kubernetes)  
(12) [https://artifacthub.io/packages/helm/minio-official/minio](https://artifacthub.io/packages/helm/minio-official/minio)  
(13) [https://min.io/docs/minio/linux/index.html?ref=con](https://min.io/docs/minio/linux/index.html?ref=con)