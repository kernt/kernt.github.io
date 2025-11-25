---
tags:
  - postgresql
  - datenbanken
  - system-administration
---

**To dump a database called mydb into a SQL-script file**

`pg_dump mydb > db.sql`

**To reload such a script into a (freshly created) database named newdb:**

`psql -d newdb -f db.sql`

**To dump a database into a custom-format archive file:**

`pg_dump -Fc mydb > db.dump`

**To dump a database into a directory-format archive:**

`pg_dump -Fd mydb -f dumpdir`

**To dump a database into a directory-format archive in parallel with 5 worker jobs:**

`pg_dump -Fd mydb -j 5 -f dumpdir`

**To reload an archive file into a (freshly created) database named newdb:**

`pg_restore -d newdb db.dump`

**To dump a single table named mytab:**

`pg_dump -t mytab mydb > db.sql`

**To dump all tables whose names start with emp in the detroit schema, except for the table named employee_log:**

`pg_dump -t 'detroit.emp*' -T detroit.employee_log mydb > db.sql`

**To dump all schemas whose names start with east or west and end in gsm, excluding any schemas whose names contain the word test:**

`pg_dump -n 'east*gsm' -n 'west*gsm' -N '*test*' mydb > db.sql`

**The same, using regular expression notation to consolidate the switches:**

`pg_dump -n '(east|west)*gsm' -N '*test*' mydb > db.sql`

**To dump all database objects except for tables whose names begin with ts_:**

`pg_dump -T 'ts_*' mydb > db.sql`

To specify an upper-case or mixed-case name in -t and related switches, you need to double-quote the name; else it will be folded to lower case (see Patterns). But double quotes are special to the shell, so in turn they must be quoted. Thus, to dump a single table with a mixed-case name, you need something like

`pg_dump -t "\"MixedCaseName\"" mydb > mytab.sql`
