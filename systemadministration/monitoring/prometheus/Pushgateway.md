---
tags:
  - prometheus
  - pushgateway
  - monitoring
---
Push a single sample into the group identified by `{job="some_job"}`

```sh
echo "some_metric 3.14" | curl --data-binary @- http://pushgateway.example.org:9091/metrics/job/some_job
```

Push something more complex into the group identified by ->{job="some_job",instance="some_instance"

```sh
cat <<EOF | curl --data-binary @- http://pushgateway.example.org:9091/metrics/job/some_job/instance/some_instance
# TYPE some_metric counter
some_metric{label="val1"} 42
# This one even has a timestamp (but beware, see below).
some_metric{label="val2"} 34 1398355504000
# TYPE another_metric gauge
# HELP another_metric Just an example.
another_metric 2398.283
EOF
```

Delete all metrics grouped by job and instance

```sh
curl -X DELETE http://pushgateway.example.org:9091/metrics/job/some_job/instance/some_instance
```


Delete all metrics grouped by job only

```sh
curl -X DELETE http://pushgateway.example.org:9091/metrics/job/some_job
```

