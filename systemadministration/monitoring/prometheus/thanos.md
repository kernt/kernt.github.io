---
tags:
  - prometheus
  - monitoring
  - cluster
  - thanos
---
# thanos Installation

Object Storage Konfigurationsdatei `/etc/prometheus/bucket.yml` mit S3

```sh
type: S3  
config:  
  bucket: "thanos"  
  endpoint: "minio.tuxcoch.de"  
  region: "<Region>"
```

oder

Attach the following policies to the role

```yaml
{  
    "Version": "2012-10-17",  
    "Statement": [  
        {  
            "Sid": "VisualEditor0",  
            "Effect": "Allow",  
            "Action": [  
                "s3:ListAccessPointsForObjectLambda",  
                "s3:ListBucketMultipartUploads",  
                "s3:ListAccessPoints",  
                "s3:ListBucketVersions",  
                "s3:ListJobs",  
                "s3:CreateBucket",  
                "s3:ListBucket",  
                "s3:ListMultiRegionAccessPoints",  
                "s3:ListMultipartUploadParts",  
                "s3:ListStorageLensConfigurations",  
                "s3:PutObject",  
                "s3:GetObject",  
                "s3:PutBucketNotification",  
                "s3:ListAllMyBuckets",  
                "s3:PutObjectRetention"  
            ],  
            "Resource": "*"  
        }  
    ]  
}
```

Sidecar service file i.e `/etc/systemd/system/sidecar.service` sample config

```sh
cat <<EOF >>/etc/systemd/system/sidecar.service  
[Unit]  
Description=Prometheus  
Wants=network-online.target  
After=network-online.target  
  
[Service]  
User=prometheus  
Group=prometheus  
Type=simple  
ExecStart=/usr/local/bin/thanos sidecar \  
    --prometheus.url=http://127.0.0.1:9090 \  
    --grpc-address=:10901 \  
    --http-address=:10902 \  
    --tsdb.path=/var/lib/prometheus/ \  
    --objstore.config-file=/etc/prometheus/bucket.yml  
  
[Install]  
WantedBy=multi-user.target  
EOF
```

```sh
sudo systemctl daemon-reload         ## Reload daemon   
sudo systemctl enable sidecar --now  ## start and enable the service.
```

## Setup Thanos Store

Erstellen data Verzeichnetes zum lokalem speichern der Metriken

```sh
mkdir /var/lib/prometheus-store/
```

Sidecar service file i.e. `/etc/systemd/system/store.service` sample config

```sh
cat <<EOF >>/etc/systemd/system/store.service  
[Unit]  
Description=Thnaos Store  
Wants=network-online.target  
After=network-online.target  
  
[Service]  
User=root  
Group=root  
Type=simple  
ExecStart=/usr/local/bin/thanos store \  
       --data-dir=/var/lib/prometheus-store/ \  
       --objstore.config-file=/etc/prometheus/bucket.yml \  
       --http-address=0.0.0.0:10906 \  
       --grpc-address=0.0.0.0:10905  
  
[Install]  
WantedBy=multi-user.target  
EOF
```

## ## Setup Thanos Query/Querier

Query service file i.e `/etc/systemd/system/query.service` sample config

```
cat <<EOF >>/etc/systemd/system/query.service  
[Unit]  
Description=Thanos Query  
Wants=network-online.target  
After=network-online.target  
  
[Service]  
User=root  
Group=root  
Type=simple  
ExecStart=/usr/local/bin/thanos query \  
     --http-address=0.0.0.0:10904 \  
     --grpc-address=0.0.0.0:10903 \  
     --endpoint=127.0.0.1:10901 \                        ## sidecar endpint  
     --endpoint=127.0.0.1:10905 \                        ## store endpoint  
     --endpoint=<2nd thanos sidecar server ip>:10901 \   ## sidecar endpint  
     --endpoint=<2nd thanos store server ip>:10905 \     ## store endpoint  
     --query.replica-label=prometheus-1  
  
[Install]  
WantedBy=multi-user.target  
EOF
```

Service file in the 2nd prometheus server for query service as given below

```sh
cat <<EOF >>/etc/systemd/system/query.service  
[Unit]  
Description=Thanos Query  
Wants=network-online.target  
After=network-online.target  
  
[Service]  
User=root  
Group=root  
Type=simple  
ExecStart=/usr/local/bin/thanos query \  
     --http-address=0.0.0.0:10904 \  
     --grpc-address=0.0.0.0:10903 \  
     --endpoint=127.0.0.1:10901 \                        ## sidecar endpint  
     --endpoint=127.0.0.1:10905 \                        ## store endpoint  
     --endpoint=<1st thanos sidecar server ip>:10901 \   ## sidecar endpint  
     --endpoint=<1st thanos store server ip>:10905 \     ## store endpoint  
     --query.replica-label=prometheus-2  
  
[Install]  
WantedBy=multi-user.target  
EOF
```

reload `systemd.` and start, enable service to star on boot

```
sudo systemctl daemon-reload   
sudo systemctl enable query --now     ## start and enable service
```

## Einrichten der Prometheus Konfigurationsdatei

Einrichtung des ersten Prometheus Servers basieren auf einem Amazon EC2

```
# my global config  
global:  
  scrape_interval: 15s   
  evaluation_interval: 15s   
  external_labels:  
    replica: prometheus-1  
  
# Alertmanager configuration  
alerting:  
  alertmanagers:  
    - static_configs:  
        - targets:   
            # - localhost:9093  
  
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.  
rule_files:  
  # - "first_rules.yml"  
  # - "second_rules.yml"  
  
# A scrape configuration containing exactly one endpoint to scrape:  
# Here it's Prometheus itself.  
scrape_configs:  
  - job_name: 'prometheus'  
    static_configs:  
      - targets: ['localhost:9090','<Prometheus 2nd Instance IP>:9090']  
  
  - job_name: "service_discovery"  
    ec2_sd_configs:  
       - region: "<Region>"  
         port: 9100  
    relabel_configs:  
       - source_labels: [__meta_ec2_tag_Prometheus]  
         regex: service-.*  
         action: keep
```

Einrichtung des zweiten Prometheus Servers basieren auf einem Amazon EC2

```
# my global config  
global:  
  scrape_interval: 15s   
  evaluation_interval: 15s   
  external_labels:  
    replica: prometheus-2  
  
# Alertmanager configuration  
alerting:  
  alertmanagers:  
    - static_configs:  
        - targets:  
            # - localhost:9093  
  
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.  
rule_files:  
  # - "first_rules.yml"  
  # - "second_rules.yml"  
  
# A scrape configuration containing exactly one endpoint to scrape:  
# Here it's Prometheus itself.  
scrape_configs:  
  - job_name: 'prometheus'  
    static_configs:  
      - targets: ['localhost:9090','<Prometheus 1st Instance IP>:9090']  
  
  - job_name: "service_discovery"  
    ec2_sd_configs:  
      - region: "<Region>"  
        port: 9100  
    relabel_configs:  
      - source_labels: [__meta_ec2_tag_Prometheus]  
        regex: service-.*  
        action: keep
```
