---
tags:
  - atlantis
  - opa
  - terraform
---
This article shows how to force specific tags and specific tag values in AWS resources with [Terraform](https://www.terraform.io/), [Atlantis](https://www.runatlantis.io/) and [Open Policy Agent](https://www.openpolicyagent.org/).

Lets suppose you have to enforce the following tags:

- Project
- Tribe
- BusinessUnit

And you also want to guaranty that BusinessUnit have the following values:

- MKT
- Risk
- Banking

The first thing I did was configure [default_tags](https://www.hashicorp.com/blog/default-tags-in-the-terraform-aws-provider) on Terraform. With default_tags I don't need to set tags inside every resource on my project, only once on providers block.

```c
provider "aws" {  
  region = "us-east-1"  
   
  default_tags {  
    tags = {  
      BusinessUnit      = "Risk"  
      Tribe             = "Cash Management"  
      Project           = "risk-api"  
    }  
  }  
}
```


In my Atlantis server I've configured two opa policies:

1. Forcing required default_tags

```json
package terraform.aws.common  
  
# Missing required tags  
deny[msg] {  
 changeset := input.resource_changes[_]  
 # Ignore if block is data   
 split(changeset.address, ".")[0] != "data"  
  
 changeset.provider_name == "registry.terraform.io/hashicorp/aws"  
 # Ignore if resource doesn't support tags_all  
 changeset.change.after.tags_all  
  
 required_tags := {"BusinessUnit", "Tribe", "Project"}  
 provided_tags := {tag | changeset.change.after.tags_all[tag]}  
 missing_tags := required_tags - provided_tags  
  
 count(missing_tags) > 0  
  
 msg := sprintf("%v is missing required tags: %v", [  
  changeset.address,  
  concat(", ", missing_tags),  
 ])  
}
```

You can test this policy [here](https://play.openpolicyagent.org/p/bviIlZrx36)

2. Forcing BusinessUnit default_tag values:

```json
package terraform.aws.common  
  
allowed_bu := [  
 "MKT",  
 "Risk",  
 "Banking"  
]  
  
valid_tag(tag, values) {  
 tag == values[_]  
}  
  
# Invalid BU tag  
deny[msg] {  
 changeset := input.resource_changes[_]  
 # Ignore if block is data  
 split(changeset.address, ".")[0] != "data"  
 # Ignore if resource doesn't support tags_all   
 changeset.change.after.tags_all  
  
 not valid_tag(changeset.change.after.tags_all.BusinessUnit, allowed_bu)  
 msg := sprintf("%v has an invalid tag: 'BusinessUnit=%v'. Valid BU are: %v", [  
  changeset.address,  
  changeset.change.after.tags_all.BusinessUnit,  
  concat(", ", sort(allowed_bu))  
 ])  
}
```

You can test this policy [here](https://play.openpolicyagent.org/p/xRFjBgWNmw)

Now lets enable [_Policy Check_](https://www.runatlantis.io/docs/policy-checking.html#how-it-works) on Atlantis:

1. Enabling the workflow using the following server configuration flag `--enable-policy-checks` or adding `enable-policy-checks: true` if you're using Atlantis server side config file.
2. Adding policy block over config.yaml

```json
policies:  
  owners:  
    users:  
      - user1  
      - user2  
      - ...  
  policy_sets:  
    - name: <POLICY-NAME>  
      path: <CODE_DIRECTORY>/<POLICY_DIR>/  
      source: local
```

I've created two different folders to organize my policies:

```sh
├── policies  
│   ├── tag-values  
│   │   └── resource_tag_values.rego  
│   └── tags  
│       └── resource_tags.rego
```

So my policy configurations are:

```
repos:  
  - id: github.com/<myorg>/<my-repo>  
    workflow: custom
policies:
  owners:
    users:  
      - admin1
      - admin2
  policy_sets:
    - name: required-tags
      path: /home/atlantis/policies/tags
      source: local
    - name: required-tag-values
      path: /home/atlantis/policies/tag-values
      source: local
workflows:
  custom:
    plan:
      steps:
        - init
        - plan
        - show
    policy_check:
      steps:
        - policy_check:  
            extra_args: ["-p /home/atlantis/policies/", "--all-namespaces"]  
...  
...
```

Now we just have to restart Atlantis.  
After that when you run Atlantis you can see a policy check stage:

```sh
Checking plan against the following policies:   
  required-tags  
  required-tag-values  
  
4 tests, 4 passed, 0 warnings, 0 failures, 0 exceptions
```

This is the output if you forget some tag:

```sh
exit status 1  
Checking plan against the following policies:   
  required-tags  
  required-tag-values  
FAIL - <redacted plan file> - terraform.aws.common - aws_ssm_parameter.ame_bff_portal_sam_ssm["API_USER"] is missing required tags: BusinessUnit  
FAIL - <redacted plan file> - terraform.aws.common - aws_ssm_parameter.ame_bff_portal_sam_ssm["API_PWD"] is missing required tags: BusinessUnit  
  
2 tests, 0 passed, 0 warnings, 2 failures, 0 exceptions
```

And this is the output if you set a different value to BusinessUnit tag:

```json
exit status 1  
Checking plan against the following policies:   
  required-tags  
  required-tag-values  
FAIL - <redacted plan file> - terraform.aws.common - aws_ssm_parameter.face_id_ssm["acesso_critical"] has an invalid tag: 'BusinessUnit=Risco'. Valid BU are: Banking, MKT, Risk  
FAIL - <redacted plan file> - terraform.aws.common - module.ame-badelse-acesso-bio.aws_sqs_queue.create_dlq[0] has an invalid tag: 'BusinessUnit=Risco'. Valid BU are: Banking, MKT, Risk  
  
2 tests, 0 passed, 0 warnings, 2 failures, 0 exceptions
```

You can't run `atlantis apply`until fix these tags, but if needed admin users have the possibility to skip policy checks. Is useful if you need to deploy an emergency fix for example.