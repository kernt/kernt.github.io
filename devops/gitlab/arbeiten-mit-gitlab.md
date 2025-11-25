---
tags:
  - gitlab
  - git
---
# Arbeiten mit Gitlab

Abgesehen von der Umstellung auf SSH können Sie auch weiterhin HTTPS verwenden, wenn Sie nichts dagegen haben, Ihr Passwort in Klartext zu Speichen.
Fügen Sie dies in Ihre `~/.netrc` ein und es wird nicht nach Ihrem Benutzernamen/Passwort gefragt (zumindest unter Linux und Mac):

```sh
machine github.com
       login <user>
       password <password>
```


## Quellen

* [implementing-gitlab-ci-dot-yml/](https://about.gitlab.com/2015/06/08/implementing-gitlab-ci-dot-yml/)
