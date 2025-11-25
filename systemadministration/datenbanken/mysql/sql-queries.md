---
tags:
  - sql
  - datenbanken
---
# #MySQL #Mariadb

**Benutzer auflisten**

`SELECT User, Host FROM mysql.user;`

**oder für den gesamten Status des Benutzers**

`ELECT User, Host, Password, password_expired FROM mysql.user;`

# #postgresql