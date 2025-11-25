---
tags:
  - development
  - devops
  - artifactory
  - repository
---
# artifactory einrichten

https://jfrog.com/de/community/download-artifactory-oss/

https://jfrog.com/de/artifactory/

https://jfrog.com/de/artifactory/?gads_network=g&cq_plac=&cq_plt=gp&gads_campaign_id=21216435747&gads_adgroup_id=160705742799&gads_extension_id=&gads_target_id=kwd-3018881139&gads_matchtype=b&gad_source=1

[getcli](https://jfrog.com/getcli/)
[Artifactory OSS Package Management](https://jfrog.com/community/download-artifactory-oss)

# Kubernetes jfrog OSS install

```sh
helm repo add jfrog https://charts.jfrog.io/
helm install -name artifactory jfrog/artifactory-oss
```