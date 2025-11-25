The **AWX Operator** simplifies the deployment and management of **Ansible AWX** in Kubernetes. While it abstracts away much of the complexity, there can still be challenges when setting it up or customizing its behavior. This article explores common troubleshooting steps and advanced configuration techniques for users seeking to optimize and extend their AWX deployment.

# Prerequisites

Before diving into troubleshooting and advanced configuration of the AWX Operator, ensure that your environment meets the following requirements:

- **AWX Operator Version:** `2.19.1`
- **Kubernetes Server Version:** `1.24.17` or higher
- **kubectl Version:** `1.30.3` or higher

These versions are critical for compatibility and stability when deploying and managing AWX with the AWX Operator. Make sure that your cluster is up-to-date and your CLI tools are properly configured before proceeding.

# Advanced Configuration Tips for AWX Operator

## Configure No Log

It is possible to show task output for debugging by setting no_log to false on the AWX CR spec. This will show the output in the `awx-operator` logs for any failed tasks where no_log was set to true.

For example:

```yaml
---  
apiVersion: awx.ansible.com/v1beta1  
kind: AWX  
metadata:  
  name: awx-demo  
spec:  
  service_type: nodeport  
  no_log: false                  # <------------
```

## General Debugging

When the operator is deploying AWX, it is running the `installer` role inside the operator container. If the AWX CR's status is `Failed`, it is often useful to look at the `awx-operator` container logs, which show the installer role's output. To see these logs, run:

kubectl logs deployments/awx-operator-controller-manager -c awx-manager -f

## Inspect k8s Resources

Past that, it is often useful to inspect various resources the AWX Operator manages like:

```sh
awx  
awxbackup  
awxrestore  
pod  
deployment  
pvc  
service  
ingress  
route  
secrets  
serviceaccount
```

And if installing via OperatorHub and OLM:

```
subscription  
csv  
installPlan  
catalogSource
```

To inspect these resources you can use these commands

```sh
# Inspecting k8s resources  
kubectl describe -n <namespace> <resource> <resource-name>  
kubectl get -n <namespace> <resource> <resource-name> -o yaml  
kubectl logs -n <namespace> <resource> <resource-name>  
  
# Inspecting Pods  
kubectl exec -it -n <namespace> <pod> <pod-name>
```

These AWX CRDs (k8s manifests) will be installed:

```
namespace/awx created  
customresourcedefinition.apiextensions.k8s.io/awxbackups.awx.ansible.com created  
customresourcedefinition.apiextensions.k8s.io/awxrestores.awx.ansible.com created  
customresourcedefinition.apiextensions.k8s.io/awxs.awx.ansible.com created  
serviceaccount/awx-operator-controller-manager created  
role.rbac.authorization.k8s.io/awx-operator-awx-manager-role created  
role.rbac.authorization.k8s.io/awx-operator-leader-election-role created  
clusterrole.rbac.authorization.k8s.io/awx-operator-metrics-reader created  
clusterrole.rbac.authorization.k8s.io/awx-operator-proxy-role created  
rolebinding.rbac.authorization.k8s.io/awx-operator-awx-manager-rolebinding created  
rolebinding.rbac.authorization.k8s.io/awx-operator-leader-election-rolebinding created  
clusterrolebinding.rbac.authorization.k8s.io/awx-operator-proxy-rolebinding created  
configmap/awx-operator-awx-manager-config created  
service/awx-operator-controller-manager-metrics-service created  
deployment.apps/awx-operator-controller-manager created
```

## Disable IPv6

I found an issue when AWX failed to start on `IPv6-disabled` environments, see details [here](https://github.com/ansible/awx-operator/issues/1017). Therefore, this configuration is necessary when starting your awx instance on the Kubernetes environment.

Starting with AWX Operator release `0.24.0`, [IPv6 was enabled in ngnix configuration](https://github.com/ansible/awx-operator/pull/950) which causes upgrades and installs to fail in environments where IPv6 is not allowed. Starting in `1.1.1` the release, you can set the `ipv6_disabled` flag on the AWX spec. If you need to use an AWX operator version between `0.24.0` and `1.1.1` in an IPv6-disabled environment, it is suggested to enable ipv6 on worker nodes.

To disable ipv6 on Nginx configuration (`awx-web` container), add the following to the AWX spec.

The following variables are customizable:

```sh
apiVersion: awx.ansible.com/v1beta1  
kind: AWX  
metadata:  
  name: awx  
spec:  
  service_type: ClusterIP  
  ipv6_disabled: true
```

## Admin user account configuration

Three variables are customizable for the admin user account creation.

1. `admin_user` default value `admin`
2. `admin_email` default value `test@example.com`
3. `admin_password_secret` default value `Empty string`

Note: `admin_password_secret` must be a Kubernetes secret and not your text clear password.

If `admin_password_secret` is not provided, the operator will look for a secret named `<resourcename>-admin-password` for the admin password. If it is not present, the operator will generate a password and create a Secret from it named `<resourcename>-admin-password`.

To retrieve the admin password, run `kubectl get secret <resourcename>-admin-password -o jsonpath="{.data.password}" | base64 --decode ; echo`

The secret that is expected to be passed should be formatted as follows:

```yaml
---  
apiVersion: v1  
kind: Secret  
metadata:  
  name: <resourcename>-admin-password  
  namespace: <target namespace>  
stringData:  
  password: mysuperlongpassword
```

The following variables are customizable in AWX:

```yaml
---  
apiVersion: awx.ansible.com/v1beta1  
kind: AWX  
metadata:  
  name: awx  
spec:  
  service_type: ClusterIP  
  admin_user: admin  
  admin_email: admin@test.com
```

## PostgreSQL Database Connections

The default postgres.conf in AWX postgres container has max_connections set to `100` (number of connections to the database). This is small and easily causes an [issue](https://github.com/ansible/awx-operator/issues/1151) if reaches this limits (`awx_database_connections_total`). In order to optimize the database, you should increase its max connections as follows:

```yaml
---  
apiVersion: awx.ansible.com/v1beta1  
kind: AWX  
metadata:  
  name: awx  
spec:  
  service_type: ClusterIP  
  postgres_extra_args:  
    - '-c'  
    - 'max_connections=1000'

another database customization example could be:
```

```yaml
---  
spec:  
  ...  
  postgres_resource_requirements:  
    requests:  
      cpu: 500m  
      memory: 2Gi  
    limits:  
      cpu: '1'  
      memory: 4Gi  
  postgres_storage_requirements:  
    requests:  
      storage: 8Gi  
    limits:  
      storage: 50Gi  
  postgres_storage_class: fast-ssd  
  postgres_extra_args:  
    - '-c'  
    - 'max_connections=1000'
```

If `postgres_storage_class` is not defined, PostgreSQL will store it's data on a volume using the default storage class for your cluster.

# Scaling AWX for High Availability

AWX can be scaled horizontally for better performance and reliability. You can scale the web and task containers independently by modifying the `spec.replicas` field in the AWX Custom Resource.

## Example: Scaling AWX Web and Task Components

```yaml
apiVersion: awx.ansible.com/v1beta1  
kind: AWX  
metadata:  
  name: awx-instance  
spec:  
  replicas: 3
```

## Scaling the Web and Task Pods independently

You can scale replicas up or down for each deployment by using the `web_replicas` or `task_replicas` respectively. You can scale all pods across both deployments by using `replicas` as well. The logic behind these CRD keys acts as such:

- If you specify the `replicas` field, the key passed will scale both the `web` and `task` replicas to the same number.
- If `web_replicas` or `task_replicas` is ever passed, it will override the existing `replicas` field on the specific deployment with the new key value.

These new replicas can be constrained in a similar manner to previous single deployments by appending the particular deployment name in front of the constraint used. More about those new constraints can be found in the [Assigning AWX pods to specific nodes](https://ansible.readthedocs.io/projects/awx-operator/en/latest/user-guide/advanced-configuration/assigning-awx-pods-to-specific-nodes.html) page.

```yaml
apiVersion: awx.ansible.com/v1beta1  
kind: AWX  
metadata:  
  name: awx-instance  
spec:  
  replicas: 3  
  web_replicas: 5  
  task_replicas: 3
```

## AWX Operator Prometheus Metrics

By default, a metrics endpoint is available in the API: `/api/v2/metrics/` that surfaces instantaneous metrics about the controller, which can be consumed by system monitoring software like the open-source project Prometheus.

The type of data shown at the `metrics/` endpoint is `Content-type: text/plain` and `application/json` as well. This endpoint contains useful information, such as counts of how many active user sessions there are, or how many jobs are actively running on each controller node. Prometheus can be configured to scrape these metrics from the controller by hitting the controller metrics endpoint and storing this data in a time-series database. Clients can later use Prometheus in conjunction with other software like Grafana or Metricsbeat to visualize that data and set up alerts.

AWX operator users can access via `https://awx-hostname/api/v2/metrics/` or `[http://awx-hostname/api/v2/metrics/](https://awx-hostname/api/v2/metrics/)`

You need to log in to view the [metrics](https://docs.ansible.com/automation-controller/4.2.0/html/administration/metrics.html), however, you can disable them via extra settings:

```yaml
---  
apiVersion: awx.ansible.com/v1beta1  
kind: AWX  
metadata:  
  name: awx  
  labels:  
    env: "dev"  
    app: "awx"  
spec:  
  replicas: 3  
  extra_settings:  
    - setting: ALLOW_METRICS_FOR_ANONYMOUS_USERS  
      value: "True"
```

# Debug Web Requests on Settings causes AWX web UI service Broken

Enabling debugging in ‘Settings’ in the AWX web UI causes the web service to break. UI crashed and didn't get to the login screen after enabling the setting. Shows; Server Error A server error has occurred.

see issue details [here](https://github.com/ansible/awx-operator/issues/1485). There is a workaround that mounting `emptyDir` to `/var/log/tower` will fix this issue.

```
spec:  
  ...  
  extra_volumes: |  
    - name: awx-web-debug  
      emptyDir: {}  
  web_extra_volume_mounts: |  
    - name: awx-web-debug  
      mountPath: "/var/log/tower"

```
an example as follows:

```yaml
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
  labels:
    env: "dev"
    app: "awx"
spec:
  replicas: 3
  extra_volumes: |
    - name: awx-web-debug
      emptyDir: {}
  web_extra_volume_mounts: |
    - name: awx-web-debug
      mountPath: "/var/log/tower"
```

