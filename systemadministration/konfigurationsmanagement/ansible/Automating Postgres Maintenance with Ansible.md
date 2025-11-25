---
tags:
  - konfigurationsmanagement
  - ansible
  - potgresql
  - vpn
  - maintenance
  - operations
---
# Ansible Postgrsql maintenance

In database management, routine maintenance is crucial to ensure optimal performance and reliability. One common issue that arises is **table bloat**, which can significantly impact query performance and overall database efficiency. In this article, we’ll explore how to automate the process of checking for table bloat in PostgreSQL and running `pg_repack` to reclaim space using Ansible.
# Why Manage Table Bloat?

Table bloat occurs when the physical size of a table increases beyond what is necessary for storing the actual data. This often results from frequent updates and deletions, leading to **dead tuples** that consume space but are not returned in query results. If left unchecked, bloat can lead to increased disk usage and slower query performance.
# The Role of pg_repack

`pg_repack` is a tool that can help combat table bloat by reorganizing tables and reclaiming wasted space without requiring downtime. It essentially rebuilds tables in the background, allowing you to maintain performance while cleaning up bloat.
# Overview of Our Automation Strategy

We will create a shell script to check for table bloat across all tables in a specified PostgreSQL database. If bloat is found to exceed a defined threshold, the script will automatically run `pg_repack` on the affected tables. We'll then write an Ansible playbook to schedule this script for execution.
# What is Ansible?

Ansible is an open-source automation tool used for configuration management, application deployment, and task automation. It allows system administrators to manage servers and applications through simple, declarative configuration files known as **playbooks**. Ansible can greatly simplify routine tasks, making it easier to maintain and deploy infrastructure.
# Installing Ansible

To get started with Ansible, you need to install it on your system. If you are using a Red Hat-based distribution like CentOS or Fedora, you can easily install Ansible using the following command:

`sudo dnf install ansible`

After installation, you can verify that Ansible is working by checking its version:

`ansible --version`
# Step 1: Create the Bloat Check and Repack Script

Create a shell script named `check_bloat_and_repack.sh`. This script will connect to the PostgreSQL database, assess table bloat, and run `pg_repack` when necessary.

```sh
#!/bin/bash
942c3da20c9b94b45f3e2b37ddb35a25fa24f93e:obivalt/konfigurationsmanagement/ansible/Automating Postgres Maintenance with Ansible.md
  
# IP and Port
IP="10.**.**.**"
PORT="1907"
  
# Bloat percentage threshold
BLOAT_THRESHOLD=20.0
  
# .pgpass file check
PGPASS_FILE="/var/lib/pgsql/.pgpass"
  
if [ ! -f "$PGPASS_FILE" ]; then
  echo "$PATRONI_IP:$PATRONI_PORT:*:postgres:password" > "$PGPASS_FILE"
  chmod 600 "$PGPASS_FILE"
fi  
```
# Database to check

`db="your_database_name"` 
# Check for bloat

```
bloat_info=$(psql -h $PATRONI_IP -p $PATRONI_PORT -U postgres -d $db -t -c "

SELECT schemaname, tblname,  
  CASE WHEN tblpages > 0 AND tblpages - est_tblpages_ff > 0  
    THEN 100 * (tblpages - est_tblpages_ff) / tblpages::float  
    ELSE 0  
  END AS bloat_pct  
FROM (  
  SELECT ceil(reltuples / ((bs - page_hdr) / tpl_size)) + ceil(toasttuples / 4) AS est_tblpages_ff,  
         tblpages, bs, tblid, schemaname, tblname, is_na  
  FROM (  
    SELECT (4 + tpl_hdr_size + tpl_data_size + (2 * ma)  
            - CASE WHEN tpl_hdr_size % ma = 0 THEN ma ELSE tpl_hdr_size % ma END  
            - CASE WHEN ceil(tpl_data_size)::int % ma = 0 THEN ma ELSE ceil(tpl_data_size)::int % ma END  
           ) AS tpl_size,  
           (heappages + toastpages) AS tblpages,  
           heappages, toastpages, reltuples, toasttuples, bs, page_hdr, tblid, schemaname, tblname, fillfactor, is_na  
    FROM (  
      SELECT tbl.oid AS tblid, ns.nspname AS schemaname, tbl.relname AS tblname, tbl.reltuples,  
             tbl.relpages AS heappages, COALESCE(toast.relpages, 0) AS toastpages,  
             COALESCE(toast.reltuples, 0) AS toasttuples,  
             COALESCE(substring(array_to_string(tbl.reloptions, ' ') FROM 'fillfactor=([0-9]+)')::smallint, 100) AS fillfactor,  
             current_setting('block_size')::numeric AS bs,  
             CASE WHEN version() ~ 'mingw32' OR version() ~ '64-bit|x86_64|ppc64|ia64|amd64' THEN 8 ELSE 4 END AS ma,  
             24 AS page_hdr,  
             23 + CASE WHEN MAX(COALESCE(s.null_frac, 0)) > 0 THEN (7 + COUNT(s.attname)) / 8 ELSE 0::int END  
               + CASE WHEN BOOL_OR(att.attname = 'oid' AND att.attnum < 0) THEN 4 ELSE 0 END AS tpl_hdr_size,  
             SUM((1 - COALESCE(s.null_frac, 0)) * COALESCE(s.avg_width, 0)) AS tpl_data_size,  
             BOOL_OR(att.atttypid = 'pg_catalog.name'::regtype) OR SUM(CASE WHEN att.attnum > 0 THEN 1 ELSE 0 END) <> COUNT(s.attname) AS is_na  
      FROM pg_attribute AS att  
      JOIN pg_class AS tbl ON att.attrelid = tbl.oid  
      JOIN pg_namespace AS ns ON ns.oid = tbl.relnamespace  
      LEFT JOIN pg_stats AS s ON s.schemaname = ns.nspname AND s.tablename = tbl.relname AND s.inherited = false AND s.attname = att.attname  
      LEFT JOIN pg_class AS toast ON tbl.reltoastrelid = toast.oid  
      WHERE NOT att.attisdropped
      AND tbl.relkind IN ('r', 'm')
      GROUP BY 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
    ) AS s
  ) AS s2
) AS s3
WHERE schemaname NOT IN ('pg_catalog', 'information_schema');
# Check bloat percentage and run pg_repack on exceeding tables
")
```
# Check bloat percentage and run pg_repack on exceeding tables

```sh
while IFS="|" read -r schemaname tblname bloat_pct; do  
    bloat_pct=$(echo $bloat_pct | xargs) # Remove leading/trailing whitespace  
    if (( $(echo "$bloat_pct > $BLOAT_THRESHOLD" | bc -l) )); then  
        echo "Significant bloat found in database $db, table $schemaname.$tblname (bloat_pct: $bloat_pct). Running pg_repack..."  
        /usr/pgsql-13/bin/pg_repack -h $PATRONI_IP -p $PATRONI_PORT -U postgres -d $db -t "$schemaname.$tblname"  
    else  
        echo "No significant bloat found in database $db, table $schemaname.$tblname (bloat_pct: $bloat_pct)."  
    fi  
done <<< "$bloat_info"
```
# Step 2: Create the Ansible Playbook

Now, we will create an Ansible playbook named `pg_repack_bloat_check.yml` to run our script on specified database servers.

```yaml
---  
- name: Check PostgreSQL Table Bloat and Run pg_repack  
  hosts: db_servers  
  become: yes  
  become_user: postgres  
  vars:  
    ip: "10.**.**.**"  
    port: "1907"  
    bloat_threshold: 20.0  
    script_file: "/var/lib/pgsql/check_bloat_and_repack.sh"  # Path to your shell script 
  tasks:  
    - name: Ensure .pgpass file exists  
      copy:  
        content: "{{ ip }}:{{ port }}:*:postgres:password"  
        dest: "/var/lib/pgsql/.pgpass"  
        mode: '0600'  
      when: not ansible_env.HOME == "/var/lib/pgsql" or not lookup('file', '/var/lib/pgsql/.pgpass')  
    - name: Execute the bloat check and pg_repack script  
      command: bash {{ script_file }}  
      register: script_output  
      ignore_errors: yes  
    - name: Display script output  
      debug:  
        var: script_output.stdout_lines
```
# Step 3: Running the Playbook

To execute the playbook, use the following command:

```sh
ansible-playbook -i inventory.ini pg_repack_bloat_check.yml
```
# Schedule with crontab

```sh
0 2 * * * /usr/bin/ansible-playbook -i /var/lib/pgsl/inventory.ini /var/lib/pgsql/pg_repack_bloat_check.yml >> /var/log/ansible_pg_repack.log 2>&1
```
# Conclusion

By following this guide, you can automate the process of checking for table bloat in your PostgreSQL databases and running `pg_repack` as needed. This not only improves performance but also ensures your databases remain efficient and responsive. With Ansible, you can further extend this automation by scheduling the playbook execution using cron jobs or other scheduling mechanisms, ensuring regular maintenance without manual intervention. For more detailed and technical articles like this, keep following our blog on Medium. If you have any questions or need further assistance, feel free to reach out in the comments below and [directly](https://www.linkedin.com/in/kemal-%C3%B6z-541baa1b9/).
