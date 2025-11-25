---
tags:
  - git
  - github
  - gitlab
  - pre-commit
---
# pre-commit Installation


**Using pip**

`pip install pre-commit`

**Hinzufügen einer Konfiguration**

`.pre-commit-config.yaml`

example

```yml
# -*- coding: utf-8 -*-
# vim: ft=yaml
---
# See https://pre-commit.com for more information
# See https://pre-commit.com/hooks.html for more hooks
default_stages: [commit]
repos:
  - repo: https://github.com/shellcheck-py/shellcheck-py
    rev: v0.8.0.4
    hooks:
      - id: shellcheck
        name: Check shell scripts with shellcheck
        files: ^.*\.(sh|bash|ksh)$
        types: []
  - repo: https://github.com/adrienverge/yamllint
    rev: v1.26.3
    hooks:
      - id: yamllint
        name: Check YAML syntax with yamllint
        args: [--strict, '.']
        always_run: true
        pass_filenames: false
  - repo: https://github.com/warpnet/salt-lint
    rev: v0.9.2
    hooks:
      - id: salt-lint
        name: Check Salt files using salt-lint
        args: [--severity]
        files: ^.*\.(sls|jinja|j2|tmpl|tst)$
```


**Install git hook skripte**

`pre-commit install`

**Test lauf**

`pre-commit run --all-files`



[pre-commit](https://pre-commit.com/#install)