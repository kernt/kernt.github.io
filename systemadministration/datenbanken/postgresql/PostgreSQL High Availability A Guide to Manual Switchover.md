![](OL6sJgfVEadkPNRV1ui5Bg.webp)

Hig availability (HA) is essential for ensuring that your PostgreSQL database remains operational and accessible even during server failures. By setting up a primary and standby node, you can maintain continuous service with minimal downtime. This guide provides a comprehensive step-by-step approach to setting up high availability for PostgreSQL and performing a manual switchover between primary and standby nodes.
# Understanding High Availability in PostgreSQL

High availability in PostgreSQL is achieved through replication, where a primary database continuously sends updates to one or more standby databases. In the event of a primary database failure, one of the standbys can be promoted to take over as the new primary, ensuring minimal disruption.
# Table of Contents

Pre-requisites.

1. Step 1: Install PostgreSQL on Node1 and Node2.
2. Step 2: Create Users and Database on Primary Node (Node1).
3. Step 3: Configure the Primary Node (Node1).
4. Step 4: Configure the Standby Node (Node2).
5. Step 5: Verify Replication Setup.
6. Step 6: Perform a Switchover.
7. Step 7: Reconfigure the Old Primary as the New Standby.
8. Conclusion.
# Pre-requisites:

Before diving into the setup of PostgreSQL high availability with manual switchover, ensure you have the following prerequisites in place:

- Two servers (Node1 and Node2) with PostgreSQL installed.
- Network connectivity between the servers.
- Basic knowledge of PostgreSQL commands and configurations.
- Appropriate permissions to execute commands on both servers.

# Step 1: Install PostgreSQL on Node1 and Node2:

To start, install PostgreSQL on both the primary (Node1) and standby (Node2) nodes.

```
# Install the PostgreSQL server packages  
sudo dnf module install postgresql:15/server -y  
  
# Initialize the database cluster  
sudo postgresql-setup --initdb  
  
# Start the postgresql service  
sudo systemctl start postgresql.service  
  
# Enable the postgresql service to start at boot  
sudo systemctl enable postgresql.service
```
# Step 2: Create Users and Database on Primary Node (Node1):

Set up necessary users and a database on the primary node.

```sh
# Connect postgres  
sudo -u postgres psql -d postgres  
  
# To show postgres password  
SELECT usename, passwd FROM pg_shadow WHERE usename = 'postgres';  
  
# Update postgres password  
ALTER USER postgres WITH PASSWORD 'password';  
  
# Create new User 'Kong'  
CREATE USER kong WITH PASSWORD 'kong'; CREATE DATABASE kong OWNER kong;   
  
# Create replicator user  
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'replicator';  
  
# Exit from postgres database  
exit
```
# Step 3: Configure the Primary Node (Node1):

Modify the configuration files on the primary node to enable replication.

```sh
# Update postgresql.conf  
sudo vim /var/lib/pgsql/data/postgresql.conf  
  
listen_addresses = '*'   
password_encryption = scram-sha-256  
wal_level = replica  
synchronous_commit = on  
max_wal_senders = 10  
synchronous_standby_names = '*'  
  
# Alternatively, you can use sed commands to update the configurations:  
sudo sed -i "s/^#listen_addresses =.*$/listen_addresses = '*'/g" /var/lib/pgsql/data/postgresql.conf  
sudo sed -i "s/^#password_encryption =.*$/password_encryption = scram-sha-256/g" /var/lib/pgsql/data/postgresql.conf  
sudo sed -i "s/^#wal_level =.*$/wal_level = replica/g" /var/lib/pgsql/data/postgresql.conf  
sudo sed -i "s/^#synchronous_commit =.*$/synchronous_commit = on/g" /var/lib/pgsql/data/postgresql.conf  
sudo sed -i "s/^#max_wal_senders =.*$/max_wal_senders = 10/g" /var/lib/pgsql/data/postgresql.conf  
sudo sed -i "s/^#synchronous_standby_names =.*$/synchronous_standby_names = '*'/g" /var/lib/pgsql/data/postgresql.conf  
  
# Update pg_hba.conf for replication  
sudo vim /var/lib/pgsql/data/pg_hba.conf  
host    all             all             [node1_ip_address]/32       scram-sha-256  
host    all             all             [node2_ip_address]/32       scram-sha-256  
host    replication     replicator      [node1_private_ip]/0        scram-sha-256  
host    replication     replicator      [node2_private_ip]/0        scram-sha-256  
  
# Restart PostgreSQL service  
sudo systemctl restart postgresql.service
```

# Step 4: Configure the Standby Node (Node2):

Prepare the standby node to replicate from the primary node.

```sh
# Stop PostgreSQL and remove existing data  
sudo systemctl stop postgresql  
sudo rm -rf /var/lib/pgsql/data/  
  
# Get backup from the primary node  
sudo -u postgres -i  
pg_basebackup -R -h [primary_node_ip] -U replicator -D /var/lib/pgsql/data -P  # enter replicator password  
exit  
  
# Update postgresql.conf  
sudo vim /var/lib/pgsql/data/postgresql.conf  
hot_standby = on  
  
# Alternatively, you can use sed commands to update the configurations:  
sudo sed -i "s/^#hot_standby =.*$/hot_standby = on/g" /var/lib/pgsql/data/postgresql.conf  
  
# Start PostgreSQL service  
sudo systemctl start postgresql.service
```
# Step 5: Verify Replication Setup:

Once the replication setup is complete, verify it on the primary node.

```sh
sudo -u postgres -i  
  
psql -c "select usename, application_name, client_addr, state, sync_priority, sync_state from pg_stat_replication;"
```
# Step 6: Perform a Switchover:

In the event of a failure or maintenance on the primary node, perform a manual switchover to promote the standby node.

```sh
# stop primary  
sudo systemctl stop postgresql.service  
sudo systemctl status postgresql.service  
  
# connect to standbt and provide read-write permission for standby  
sudo -u postgres psql -d kong  
  
SELECT pg_promote();  
SELECT pg_is_in_recovery();  
exit  
  
# change standby config  
sudo vim /var/lib/pgsql/data/postgresql.conf  
  
synchronous_standby_names = ''   
hot_standby = off  
synchronous_commit = off  
  
# restart   
sudo systemctl start postgresql
```

# Step 7: Reconfigure the Old Primary as the New Standby:

To maintain high availability, reconfigure the old primary node to act as the new standby.

```sh
# On the new standby (old primary)  
sudo -u postgres -i  
PGPASSWORD='replicator' pg_rewind -D /var/lib/pgsql/data --source-server='host=[new_primary_ip] port=5432 user=replicator dbname=kong' -P  
exit  
  
# Update postgresql.conf  
sudo vim /var/lib/pgsql/data/postgresql.conf  
synchronous_standby_names = '*'   
hot_standby = on  
synchronous_commit = on  
  
# Alternatively, you can use sed commands to update the configurations:  
sudo sed -i "s/^synchronous_commit =.*$/synchronous_commit = on/g" /var/lib/pgsql/data/postgresql.conf  
sudo sed -i "s/^hot_standby =.*$/hot_standby = on/g" /var/lib/pgsql/data/postgresql.conf  
sudo sed -i "s/^synchronous_standby_names =.*$/synchronous_standby_names = '*'/g" /var/lib/pgsql/data/postgresql.conf  
  
# Start PostgreSQL service on the new standby  
sudo systemctl start postgresql  
  
# On the new primary, grant necessary permissions  
sudo -u postgres psql -d kong  
SELECT proname, proargtypes FROM pg_proc WHERE proname = 'pg_read_binary_file';  
GRANT EXECUTE ON FUNCTION pg_read_binary_file(text, bigint, bigint, boolean) TO replicator;  
SELECT proname, pg_get_function_arguments(oid) AS arguments FROM pg_proc WHERE proname = 'pg_ls_dir';  
GRANT EXECUTE ON FUNCTION pg_ls_dir(text, boolean, boolean) TO replicator;  
SELECT proname, pg_get_function_arguments(oid) AS arguments FROM pg_proc WHERE proname = 'pg_stat_file';  
GRANT EXECUTE ON FUNCTION pg_stat_file(text, boolean) TO replicator;
```

https://technotim.live/posts/postgresql-high-availability/