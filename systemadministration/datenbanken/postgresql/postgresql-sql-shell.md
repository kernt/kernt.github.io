---
tags:
  - postgresql
  - datenbanken
---
# postgresql-sql-shell

**Anmeldung an der Datenbank**

`su - postgres`

**Eine Datenbank anlegen**

```sql
CREATE DATABASE db_name
OWNER =  role_name
TEMPLATE = template
ENCODING = encoding
LC_COLLATE = collate
LC_CTYPE = ctype
TABLESPACE = tablespace_name
CONNECTION LIMIT = max_concurrent_connection
```

**Datenbanken auflisten**

`\1`

**Tabelle _employees_ erstellen**

`CREATE TABLE employees (employee_id int, first_name varchar, last_name varchar);`

**View vom Inhalt _employees_**

`SELECT * FROM employees;`

**Auflisten von tabelen in einer DB**

`\dt`

**Tabelen löschen**

`DROP TABLE employees;`

**Spalten hinzufügen**

`ALTER TABLE employees ADD start_date date;`

**Zeilen hinzufügen**

`INSERT INTO employees VALUES (2, 'Jane', 'Smith', '2015-03-09');`

**Zeilen modifiezren**

`UPDATE employees SET start_date = '2016-09-28' WHERE employee_id = '1';`

**Query einer tabelle**

`SELECT last_name,employee_id FROM employees;`

**Role anlegen**

`createuser examplerole --pwprompt`

**Einer role Berechtigungen  geben**

`GRANT ALL ON employees TO examplerole;`

**Alle rolen anzeigen**

`\du`

Quelle:

[postgresql-create-database](https://www.guru99.com/postgresql-create-database.html)
