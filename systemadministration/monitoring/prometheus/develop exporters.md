---
tags:
  - monitoring
  - prometheus
---
# Build exporter mit go

## GO und Package Dependencies

## Erstellen eines Verzeichnises  “my_first_exporter”

```go
go mod init my_first_exporter
go get github.com/prometheus/client_golang
go get github.com/joho/godotenv--> creates go.mod file
--> Installs dependency into the go.mod file
```

## Create main.go file, and paste in the following

```go
package main

import (
 "github.com/joho/godotenv"
 "github.com/prometheus/client_golang/prometheus"
 "github.com/prometheus/client_golang/prometheus/promhttp"
)
```

## Put in Entry-point function main()

```go
func main() {
}
```

## Add prometheus metrics endpoint and listen on the server port

```go
func main() {
   http.Handle("/metrics", promhttp.Handler())
   log.Fatal(http.ListenAndServe(":9101", nil))
}
```


Explore External Services API with curl.
The application I am monitoring is MirthConnect. I will be making two API calls:
a. get channel statistics  
b. get channel id and name mapping

```sh
curl -k --location --request GET 'https://apihost/api/channels/statistics' \
--user admin:admin
  
curl -k --location --request GET 'https://apihost/api/channels/idsAndNames' \
--user admin:admin
```

Convert the curl call into go http calls, and handle the result parsing

This is the most difficult step in the entire process if you are new to Go. For my example, my endpoints return XML payload. This means I had to deserialize XML with the **“encoding/xml”** package.

A successful conversion means my GO program can perform the same API calls as the curl commands. Here are some GO packages to read up about:

**“crypto/tls" = specify TLS connection options.
“io/ioutil” = reading the result payload from buffer into strings
“net/http” = create transport and clients
“strconv” = converting string to numbers like floating point/integer**


You can iterate through the payload and print out their values with the **“log”** package.

Declare Prometheus metric descriptions

In Prometheus, each metric is made of the following: metric name/metric label values/metric help text/metric type/measurement.

Example:

**HELP** promhttp_metric_handler_requests_total Total number of scrapes by HTTP status code. 

**TYPE** promhttp_metric_handler_requests_total **counter**  
promhttp_metric_handler_requests_total{**code**=”200"} **1.829159e+06**  
promhttp_metric_handler_requests_total{**code**=”500"} **0**  
promhttp_metric_handler_requests_total{**code**=”503"} **0**

For application scrapers, we will define Prometheus metric descriptions, which includes metric name/metric labels/metric help text.

```go
messagesReceived = prometheus.NewDesc(
 prometheus.BuildFQName(namespace, "", "messages_received_total"),
 "How many messages have been received (per channel).",
 []string{"channel"}, nil,
)
```


# Build expoter with Phyton

Use Phyton to make a exporter

```python 
Let’s see a small but complete exporter implemented in Python.
> notes and explanation in comments and below the snippet:

exporter.py

```py
"""Application exporter"""

import os
import time
from prometheus_client import start_http_server, Gauge, Enum
import requests

class AppMetrics:
    """
    Representation of Prometheus metrics and loop to fetch and transform
    application metrics into Prometheus metrics.
    """

    def __init__(self, app_port=80, polling_interval_seconds=5):
        self.app_port = app_port
        self.polling_interval_seconds = polling_interval_seconds

        # Prometheus metrics to collect
        self.current_requests = Gauge("app_requests_current", "Current requests")
        self.pending_requests = Gauge("app_requests_pending", "Pending requests")
        self.total_uptime = Gauge("app_uptime", "Uptime")
        self.health = Enum("app_health", "Health", states=["healthy", "unhealthy"])

    def run_metrics_loop(self):
        """Metrics fetching loop"""

        while True:
            self.fetch()
            time.sleep(self.polling_interval_seconds)

    def fetch(self):
        """
        Get metrics from application and refresh Prometheus metrics with
        new values.
        """

        # Fetch raw status data from the application
        resp = requests.get(url=f"http://localhost:{self.app_port}/status")
        status_data = resp.json()

        # Update Prometheus metrics with application metrics
        self.current_requests.set(status_data["current_requests"])
        self.pending_requests.set(status_data["pending_requests"])
        self.total_uptime.set(status_data["total_uptime"])
        self.health.state(status_data["health"])

def main():
    """Main entry point"""

    polling_interval_seconds = int(os.getenv("POLLING_INTERVAL_SECONDS", "5"))
    app_port = int(os.getenv("APP_PORT", "80"))
    exporter_port = int(os.getenv("EXPORTER_PORT", "9877"))

    app_metrics = AppMetrics(
        app_port=app_port,
        polling_interval_seconds=polling_interval_seconds
    )
    start_http_server(exporter_port)
    app_metrics.run_metrics_loop()

if __name__ == "__main__":
    main()
```


# [exporter-toolkit](https://github.com/prometheus/exporter-toolkit/tree/master)


