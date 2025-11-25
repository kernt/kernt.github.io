---
tags:
  - logs
  - logging
  - container
  - prometheus
  - elasticsearch
---
Monitoring and logging are crucial for maintaining a reliable system. Whether you’re overseeing cloud infrastructure, microservices, or CI/CD pipelines, automation scripts provide a strong foundation to avoid late-night troubleshooting of production issues. In this article, we’ll dive into 15 must-know scripts for every DevOps engineer. These powerful yet simple tools can help you monitor system performance, spot bottlenecks, and keep logs organized.

This guide is beginner-friendly, but even seasoned DevOps professionals will find valuable insights. Let’s explore the significance of each script, how to implement them, and real-world scenarios where they can make a difference!
# 1. Disk Space Monitoring Script

**Why it’s important:** Running out of disk space can cause application crashes, failed builds, or even data loss.

**How it works:** A Bash script checks disk usage across your servers and sends an alert if usage exceeds a defined threshold.

**Sample Script:**

```sh
#!/bin/bash  
THRESHOLD=80  
df -h | awk '{if($5+0 > '$THRESHOLD') print $0}'
```

**Real-world scenario:** A web server hosting a high-traffic e-commerce site during a holiday sale generates rapid log growth. This script provides an early warning, allowing space to be cleared or storage increased before issues arise.
# 2. CPU Usage Monitoring Script

**Why it’s important:** High CPU usage can indicate that your application isn’t scaling properly or that a rogue process is hogging resources.

**How it works:** A Python script uses **psutil** to monitor CPU usage and sends alerts when it exceeds a threshold.

**Sample Script:**

```Python
import psutil  
import smtplib  
  
if psutil.cpu_percent(interval=1) > 80:  
    print("High CPU Usage Alert!")
```

**Real-world scenario:** During a sudden traffic spike in a microservice-based system, this script pinpoints the overloaded service, helping you optimize or scale that service.
# 3. Log Rotation Automation Script

**Why it’s important:**
Large logs can quickly consume storage and become unmanageable. Automating log rotation helps keep them under control.

**How it works:**
A cron job paired with a Bash script compresses and archives old logs, while retaining recent ones.

**Sample Script:**

```sh
#!/bin/bash  
LOG_DIR="/var/log/app/"  
tar -czf $LOG_DIR/archive_$(date +%F).tar.gz $LOG_DIR/*.log  
rm $LOG_DIR/*.log
```

**Real-world scenario:** A payment gateway application generates logs every second. This script ensures old logs are archived, preventing disk space from being exhausted while keeping recent logs for troubleshooting.
# 4. Service Uptime Monitoring Script

**Why it’s important:** Immediate detection of service downtime is crucial for maintaining application reliability.

**How it works:** A Python script pings a service endpoint and sends alerts if it’s unreachable.

**Sample Script:**

```Python
import requests
try:
    response = requests.get("http://your-service-url.com")
    if response.status_code != 200:
        print("Service Down Alert!")
except Exception as e:
    print("Service Unreachable!")
```

**Real-world scenario:** A SaaS authentication service goes down. This script detects the issue swiftly, enabling you to resolve it before customers notice.
# 5. Memory Usage Monitoring Script

**Why it’s important:** Low memory can lead to application crashes and service disruptions.

**How it works:** This script checks memory usage and alerts when available memory drops below a specified threshold.

**Sample Script:**

```sh
#!/bin/bash  
THRESHOLD=500 # in MB  
AVAILABLE=$(free -m | awk '/Mem/ {print $7}')  
if [ $AVAILABLE -lt $THRESHOLD ]; then  
    echo "Low Memory Alert!"  
fi
```

**Real-world scenario:** A CI/CD pipeline crashes due to an out-of-memory error. This script provides an early warning, enabling resource allocation before failure occurs.
# 6. Docker Container Health Check Script

**Why it’s important:** Container health is crucial for ensuring the smooth operation of microservices.
**How it works:** This script checks the health of running Docker containers.
**Sample Script:**

```sh
#!/bin/bash  
for container in $(docker ps --format "{{.Names}}"); do  
    STATUS=$(docker inspect --format '{{.State.Health.Status}}' $container)  
    echo "Container: $container - Status: $STATUS"  
done
```

**Real-world scenario:** A Kubernetes deployment with multiple pods may experience container failures. This script quickly identifies unhealthy containers for faster recovery.
# 7. Kubernetes Pod Monitoring Script

**Why it’s important:** Pods are the smallest deployable units in Kubernetes. Monitoring their status ensures applications are running smoothly.

**How it works:** This script uses **kubectl** commands to check pod statuses.

**Sample Script:**

```sh
#!/bin/bash  
kubectl get pods --all-namespaces | grep -v 'Running'
```

**Real-world scenario:** A retail app on Kubernetes experiences pod failures during a sales event. This script identifies the failing pods, allowing for a quicker resolution.

# 8. Application Log Parsing Script

**Why it’s important:** Log parsing helps detect hidden issues, but manual filtering can be time-consuming.

**How it works:** A Bash script filters application logs for error messages or specific patterns.

**Sample Script:**

```sh
#!/bin/bash  
LOG_FILE="/var/log/app.log"  
grep -i "error" $LOG_FILE
```

**Real-world scenario:** A user reports intermittent errors. This script extracts error messages from logs, aiding in faster issue resolution.
# 9. Website Availability Check Script

**Why it’s important:** If your website goes offline, it’s critical to detect the outage promptly.

**How it works:** A Python script checks if your website is up and alerts if it’s down.

**Sample Script:**

```Python
import requests
try:
    response = requests.get("http://example.com")
    print("Website is up!") if response.status_code == 200 else print("Website is down!")
except:
    print("Website Unreachable!")
```
# 10. Elasticsearch Log Monitoring Script

**Why it’s important:** Elasticsearch efficiently stores logs, but querying and monitoring indices should be automated.

**Sample Script:**

```Python
import requests  
from datetime import datetime, timedelta  
  
ELASTICSEARCH_URL = 'http://els1.tuxcosch.de:9200'  
INDEX_NAME = 'logs'  
  
def get_logs():  
    end_time = datetime.now()  
    start_time = end_time - timedelta(hours=1)  
    query = {  
        "query": {  
            "bool": {  
                "must": [  
                    {"match": {"message": "HTTP 500"}}  
                ],  
                "filter": {  
                    "range": {  
                        "@timestamp": {  
                            "gte": start_time.isoformat(),  
                            "lte": end_time.isoformat()  
                        }  
                    }  
                }  
            }  
        }  
    }  
    response = requests.get(f'{ELASTICSEARCH_URL}/{INDEX_NAME}/_search', json=query)  
    response.raise_for_status()  
    return response.json()  
  
def parse_logs(log_data):  
    hits = log_data['hits']['hits']  
    for hit in hits:  
        log_message = hit['_source'].get('message', 'No message')  
        timestamp = hit['_source']['@timestamp']  
        print(f'{timestamp}: {log_message}')  
  
if __name__ == "__main__":  
    log_data = get_logs()  
    parse_logs(log_data)
```

# 11. Automated Alert Manager Script

**Why it’s important:** Alerts from Prometheus can be sent to Slack automatically, ensuring timely responses.

**Sample Script:**

```Python
import requests  
import json  
  
PROMETHEUS_URL = 'http://localhost:9090'  
SLACK_WEBHOOK_URL = 'https://hooks.slack.com/services/your/webhook/url'  
QUERY = 'ALERTS{alertstate="firing"}'  
def get_prometheus_alerts():  
    response = requests.get(f'{PROMETHEUS_URL}/api/v1/query', params={'query': QUERY})  
    return response.json()  
def send_slack_alert(alert):  
    message = {'text': f"Alert: {alert['labels']['alertname']}\nSeverity: {alert['labels']['severity']}\nDescription: {alert['annotations']['description']}"}  
    requests.post(SLACK_WEBHOOK_URL, data=json.dumps(message), headers={'Content-Type': 'application/json'})  
if __name__ == "__main__":  
    alerts = get_prometheus_alerts()['data']['result']  
    for alert in alerts:  
        send_slack_alert(alert)
```

# 12. Script for Cleaning Temp Files

**Why it’s important:** Temporary files can clutter the filesystem, leading to disk space issues.

**Sample Script:**

```Python
import os
import shutil
import time
  
TEMP_DIR = '/tmp'
AGE_THRESHOLD = 7 * 24 * 60 * 60  # 7 days in seconds
  
def clean_temp_files():
    now = time.time()
    for filename in os.listdir(TEMP_DIR):
        file_path = os.path.join(TEMP_DIR, filename)  
        if os.stat(file_path).st_mtime < now - AGE_THRESHOLD:
            if os.path.isfile(file_path) or os.path.islink(file_path):
                os.remove(file_path)
                print(f'Removed file: {file_path}')
            elif os.path.isdir(file_path):
                shutil.rmtree(file_path)
                print(f'Removed directory: {file_path}')
if __name__ == "__main__":
    clean_temp_files()
```

# 13. Script to Track DNS Resolution Time

**Why it’s important:** Monitoring DNS resolution speeds helps troubleshoot slow application performance.

**Sample Script:**

```sh
#!/bin/bash  
DNS_SERVER="8.8.8.8"  
DOMAIN="example.com"  
dig @$DNS_SERVER $DOMAIN +stats | grep "Query time"
```
# 14. Automated Backup Verification Script

**Why it’s important:** Backups must be verified to ensure they’re available in the event of a system failure.

**Sample Script:**

```sh
#!/bin/bash  
BACKUP_DIR="/backups"  
for file in $BACKUP_DIR/*.tar.gz; do  
    if ! tar -tzf $file > /dev/null; then  
        echo "Corrupted backup file: $file"  
    fi  
done
```
# 15. Cloud Cost Monitoring Script

**Why it’s important:** Managing cloud costs ensures efficient resource allocation and prevents unexpected bill spikes.

**Sample Script:**

```sh
#!/bin/bash  
aws ce get-cost-and-usage --time-period Start=$(date +%Y-%m-%d),End=$(date +%Y-%m-%d) --granularity DAILY --metrics "BlendedCost"
```