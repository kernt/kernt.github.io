---
tags:
  - jenkins
  - gitlab
  - token
---
## Jenkins Berechtigungen

Für den [Persönlichen  Token](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html#create-a-personal-access-token) werde mindestens folgende Berechtigungen benötigt
+ api
+ read_api
+ read_user
+ read_repository

Für einen [Projekt Token](https://docs.gitlab.com/ee/user/project/settings/project_access_tokens.html#create-a-project-access-token)

+ api
+ read_user
+ read_repository
+ write_repository
+ read_registry
+ read_api

[Token scops](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html#personal-access-token-scopes)