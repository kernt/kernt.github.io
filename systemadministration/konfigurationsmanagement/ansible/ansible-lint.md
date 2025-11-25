---
tags:
  - konfigurationsmanagement
  - ansible
  - ansible-lint
---

# Ansible Lint

## Ansible lint Regeln zum ausschließen

Möchte man, dass bestimmte Regeln bei der Prüfung mit **ansible-lint** ausgeschloßen sind, so kann man diese per `-x <REGEL>` angeben.

Bsp.: `ansible-lint -x ANSIBLE0011 playbook.yml`

Man kann auch direkt im Playbook bestimmte Abschnitte entsprechend markieren, damit diese nicht von **ansible-lint** geprüft werden.  
Dies ist z.B. sinnvoll wenn es sich um **False Positives** handelt.

Die entsprechende Markierung ist hierbei: `# noqa <Regel>`  
Bsp.: `# noqa 501 201`

Dies sieht dann im Playbook beispielsweise so aus:

```
- name: this would typically fire GitHasVersionRule 401 and BecomeUserWithoutBecomeRule 501
  become_user: alice  # noqa 401 501
  git: src=/path/to/git/repo dest=checkout
```

Man kann auch eigene Regel definieren, entsprechende Infos dazu findet man in der [Dokumentation](https://docs.ansible.com/ansible-lint/rules/rules.html#creating-custom-rules)

_Ansible Lint Example für Azure und Ansible_

```yaml
---
yaml-files:
  - "*.yaml"
  - "*.yml"
  - .yamllint

ignore:
  - .azure/
  - docs/
  - plugins/
  - _*.yml
  - _*.yaml

rules:
  line-length: disable
  anchors: enable
  braces:
    min-spaces-inside: 0
    max-spaces-inside: 1
  brackets:
    max-spaces-inside: 1
    max-spaces-inside-empty: 1
  colons: enable
  commas: enable
  comments:
    min-spaces-from-content: 1
    level: warning
  comments-indentation: disable
  document-end: disable
  document-start: disable
  empty-lines: enable
  empty-values: disable
  float-values: disable
  hyphens: enable
  indentation:
    level: warning
  key-duplicates: enable
  key-ordering: disable
  new-line-at-end-of-file: enable
  new-lines: enable
  octal-values:
    forbid-implicit-octal: true
    forbid-explicit-octal: true
  quoted-strings:
    quote-type: double
    required: only-when-needed
    extra-allowed:
      - '^.*({|}).*$'
      - '(\^|\$|\.\*|\+)'
  trailing-spaces:
    ignore: |
      .azure
  truthy:
    level: warning

```

_Ansible example Prod Profile und azure_

```yaml
---
# null, min, basic, moderate,safety, shared, production
profile: production

exclude_paths:
  - .cache/
  - docs/
  - project_layout/
  - .azure/
  - .vscode
  - plugins/

warn_list:
  - skip_this_tag
  - line-length
  - var-naming

# Enable checking of loop variable prefixes in roles
loop_var_prefix: "^(__|{role}_)"

```

### weitere Infos

weiterführende Informationen findet man in der [Ansible Lint Documentation](https://docs.ansible.com/ansible-lint/)