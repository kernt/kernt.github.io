---
tags:
  - datenbanken
  - influxdb
---
# Install Time Series Database, InfluxDB.

[1] 	Install InfluxDB and client tools.
root@dlp:~# apt -y install influxdb influxdb-client
[2] 	Configure InfluxDB.
By default, authentication is disabled, all users can use InfluxDB with privileges. So enable authentication first.
# create an admin user

# [admin] ⇒ specify any username you like

# [adminpassword] ⇒ set any password

root@dlp:~# influx -execute "create user admin with password 'adminpassword' with all privileges"

`root@dlp:~# influx -execute "show users"`

```sh
user  admin
----  -----
admin true

root@dlp:~# vi /etc/influxdb/influxdb.conf

[http]
  # Determines whether HTTP endpoint is enabled.
  # enabled = true

  # The bind address used by the HTTP service.
  # bind-address = ":8086"

  # Determines whether user authentication is enabled over HTTP/HTTPS.
  # line 226 : uncomment and change
  auth-enabled = true

root@dlp:~# systemctl restart influxdb
```

[3] 	After enabling authentication, access InfluxDB CLI like follows.

```
# run InfluxDB CLI and authenticate as an user

root@dlp:~# influx

Connected to http://localhost:8086 version 1.6.7~rc0
InfluxDB shell version: 1.6.7~rc0
> auth
username: admin
password:

> exit

# add user info on CLI

root@dlp:~# influx -username admin -password adminpassword

Connected to http://localhost:8086 version 1.6.7~rc0
InfluxDB shell version: 1.6.7~rc0
> exit

# set user info on environment variables

root@dlp:~# export INFLUX_USERNAME=admin
root@dlp:~# export INFLUX_PASSWORD=adminpassword
root@dlp:~# influx

Connected to http://localhost:8086 version 1.6.7~rc0
InfluxDB shell version: 1.6.7~rc0
> exit

# authenticate on HTTP API

root@dlp:~# curl -G http://localhost:8086/query?pretty=true -u admin:adminpassword --data-urlencode "q=show users"

{
    "results": [
        {
            "statement_id": 0,
            "series": [
                {
                    "columns": [
                        "user",
                        "admin"
                    ],
                    "values": [
                        [
                            "admin",
                            true
                        ]
                    ]
                }
            ]
        }
    ]
}
```

# InfluxDB : Basic User Management


This is InfluxDB Basic User Management example.
Add or delete InfluxDB users.

```sh
root@dlp:~# influx -username admin -password adminpassword

Connected to http://localhost:8086 version 1.6.7~rc0
InfluxDB shell version: 1.6.7~rc0

# create [debian] user
# replace [userpassword] to any password
> create user debian with password 'userpassword' 

# create [serverworld] user with admin privilege
> create user serverworld with password 'userpassword' with all privileges 

> show users 
user        admin
----        -----
admin       true
debian      false
serverworld true

# add admin privilege to [debian] user
> grant all privileges to "debian" 

> show users 
user        admin
----        -----
admin       true
debian      true
serverworld true

# remove admin privilege from [debian] user
> revoke all privileges from "debian" 

# change [debian] user's password
> set password for "debian" = 'newpassword' 

# delete [debian] user
> drop user "debian" 

> exit

```

# InfluxDB : Basic Database Management

This is InfluxDB Basic Database Management example.
Create Database. 

```sh
root@dlp:~# influx -username admin -password adminpassword

Connected to http://localhost:8086 version 1.6.7~rc0
InfluxDB shell version: 1.6.7~rc0

# create [test_database] database
> create database test_database 

> show databases 
name: databases
name
----
_internal
test_database

> show users 
user        admin
----        -----
admin       true
debian      false
serverworld true

# add all privileges to [serverworld] user on [test_database] database
> grant all on "test_database" to "serverworld" 

# add [read] privilege to [debian] user on [test_database] database
> grant read on "test_database" to "debian" 

> show grants for "serverworld" 
database      privilege
--------      ---------
test_database ALL PRIVILEGES

> exit

```

**Insert data to database.**

```sh
# InfluxDB data series
# ⇒ measurement,tag_set field_set timestamp
# insert <measurement>[,<tag_key>=<tag_value>[,<tag_key>=<tag_value>]] <field_key>=<field_value>[,<field_key>=<field_value>] [<timestamp>]
#
# [tag_set] is optional
# [timestamp] is optional ⇒ if [timestamp] is not specified, it is used current UNIX Time (nanosecond)
# possible to see UNIX Time (nanosecond) with [date] command ⇒ $ date +%s%N

# connect to [test_database]

root@dlp:~# influx -username serverworld -password userpassword -database test_database

Connected to http://localhost:8086 version 1.6.7~rc0
InfluxDB shell version: 1.6.7~rc0

# insert data to [cpu] measurement
> insert cpu idle=99.50
> insert cpu idle=99.00
> insert cpu idle=99.30
> select * from cpu
name: cpu
time                idle
----                ----
1688601672845748376 99.5
1688601676926901128 99
1688601680775107208 99.3

# insert data to [weather] measurement
> insert weather,location=hiroshima temperature=20
> insert weather,location=hiroshima temperature=22
> insert weather,location=osaka temperature=18
> insert weather,location=osaka temperature=19
> select * from weather
name: weather
time                location  temperature
----                --------  -----------
1688601708746672792 hiroshima 20
1688601713339254836 hiroshima 22
1688601717819722751 osaka     18
1688601722868089561 osaka     19

# possible to search data with [WHERE]
> select * from weather where "location" = 'hiroshima' and temperature <= 20
name: weather
time                location  temperature
----                --------  -----------
1688601708746672792 hiroshima 20

# show [timestamp] with RFC3339 style
> precision rfc3339
> select * from weather
name: weather
time                           location  temperature
----                           --------  -----------
2023-07-06T00:01:48.746672792Z hiroshima 20
2023-07-06T00:01:53.339254836Z hiroshima 22
2023-07-06T00:01:57.819722751Z osaka     18
2023-07-06T00:02:02.868089561Z osaka     19

> exit

```

# Delete data

```sh
# connect with RFC3339 style

root@dlp:~# influx -username serverworld -password userpassword -database test_database -precision rfc3339

Connected to http://localhost:8086 version 1.6.7~rc0
InfluxDB shell version: 1.6.7~rc0

> select * from weather
name: weather
time                           location  temperature
----                           --------  -----------
2023-07-06T00:01:48.746672792Z hiroshima 20
2023-07-06T00:01:53.339254836Z hiroshima 22
2023-07-06T00:01:57.819722751Z osaka     18
2023-07-06T00:02:02.868089561Z osaka     19

# delete data that [timestamp] are older than [2023-07-06 00:01:50]
> delete from weather where time <= '2023-07-06 00:01:50'
> select * from weather
name: weather
time                           location  temperature
----                           --------  -----------
2023-07-06T00:01:53.339254836Z hiroshima 22
2023-07-06T00:01:57.819722751Z osaka     18
2023-07-06T00:02:02.868089561Z osaka     19

# delete data that tags are [location = hiroshima]
> drop series from "weather" where "location" = 'hiroshima'

# * delete *** ⇒ not drop series from Index, possible to search with Time Interval
# * drop *** ⇒ drop series from Index, impossible to search Time Interval

# delete [measurement]
> drop measurement "cpu"
> show measurements
name: measurements
name
----
weather

# delete database
> drop database "test_database"
> show databases
name: databases
name
----
_internal

> exit

```

# InfluxDB : Set Retention Policy

This is how to create and set Retention Policy to Database.
It's possible to set retention duration on each database.

Create and Set Retention Policy. 

```sh
root@dlp:~#

influx -username serverworld -password userpassword -database test_database -precision rfc3339

Connected to http://localhost:8086 version 1.6.7~rc0
InfluxDB shell version: 1.6.7~rc0

# Syntax
# - CREATE RETENTION POLICY <policy_name> ON <database> DURATION <duration> REPLICATION <n> [SHARD DURATION <duration>] [DEFAULT]
#
# DURATION ⇒ retention duration (minimum duration is 1h)
# - unit time
#   - w : week
#   - d : day
#   - h : hour
#
# REPLICATION ⇒ number of data nodes in cluster
#   - set [1] on single node
#   - clustering is not supported on InfluxDB OSS 1.7
#
# SHARD DURATION ⇒ time range covered by a shard group (optional)

# show retention policy
# [autogen] is the default policy that has infinite retention
> show retention policies 
name    duration shardGroupDuration replicaN default
----    -------- ------------------ -------- -------
autogen 0s       168h0m0s           1        true

# create a retention policy which has 1 day retention
> create retention policy "one_day" on "test_database" duration 1d replication 1 

> show retention policies 
name    duration shardGroupDuration replicaN default
----    -------- ------------------ -------- -------
autogen 0s       168h0m0s           1        true
one_day 24h0m0s  1h0m0s             1        false

# set the new policy default
> alter retention policy "one_day" on "test_database" default 

> show retention policies 
name    duration shardGroupDuration replicaN default
----    -------- ------------------ -------- -------
autogen 0s       168h0m0s           1        false
one_day 24h0m0s  1h0m0s             1        true

# change retention for a policy
> alter retention policy "one_day" on "test_database" duration 1w 

> show retention policies 
name    duration shardGroupDuration replicaN default
----    -------- ------------------ -------- -------
autogen 0s       168h0m0s           1        false
one_day 168h0m0s 1h0m0s             1        true

> exit
```


# InfluxDB : Enable HTTPS

Enable HTTPS on InfluxDB server.
(used HTTP connection by default)

Get SSL Certificate, or Create self-signed Certificate.
It uses self signed Certificate on this example.

Enable HTTPS. 

```sh
root@dlp:~# cp /etc/ssl/private/{server.crt,server.key} /etc/influxdb/

root@dlp:~# chown influxdb:influxdb /etc/influxdb/{server.crt,server.key}

root@dlp:~# vi /etc/influxdb/influxdb.conf

# line 257 : uncomment and change
https-enabled = true

# line 260 : uncomment and change to your certificate
https-certificate = "/etc/influxdb/server.crt"

# line 263 : uncomment and change to your certificate
https-private-key = "/etc/influxdb/server.key"

root@dlp:~# systemctl restart influxdb 
```

Connect to InfluxDB from Clients via HTTPS like follows.

```sh
# on CLI connection, add [ssl] option
# if using valid certificate, specify the same hostname registered in certificate

root@dlp:~# influx -host dlp.srv.world -ssl

Connected to https://dlp.srv.world:8086 version 1.6.7~rc0
InfluxDB shell version: 1.6.7~rc0
> exit

# for self signed certificate, add [-unsafeSsl] option

root@dlp:~# influx -ssl -unsafeSsl

Connected to https://localhost:8086 version 1.6.7~rc0
InfluxDB shell version: 1.6.7~rc0
> exit

# HTTP API

root@dlp:~# curl -G https://dlp.srv.world:8086/query?pretty=true -u admin:adminpassword --data-urlencode "q=show users"

{
    "results": [
        {
            "statement_id": 0,
            "series": [
.....
.....

# HTTP API (self signed)

root@dlp:~# curl -k -G https://localhost:8086/query?pretty=true -u admin:adminpassword --data-urlencode "q=show users"

{
    "results": [
        {
            "statement_id": 0,
            "series": [
.....
.....

```

# InfluxDB : Backup and Restore

 	
This is how to back up and restore InfluxDB database.
Back up a database. 

```sh
# backup [test_database] to [/home/influxd_backup]

root@dlp:~# influxd backup -database test_database /home/influxd_backup

2023/07/05 19:17:06 backing up metastore to /home/influxd_backup/meta.00
2023/07/05 19:17:06 backing up db=test_database
2023/07/05 19:17:06 backing up db=test_database rp=autogen shard=4 to /home/influxd_backup/test_database.autogen.00004.00 since 0001-01-01T00:00:00Z
2023/07/05 19:17:06 backup complete:
2023/07/05 19:17:06     /home/influxd_backup/home/influxd_backup/meta.00
2023/07/05 19:17:06     /home/influxd_backup/home/influxd_backup/test_database.autogen.00004.00

root@dlp:~# ll /home/influxd_backup

total 8
-rw-r--r-- 1 root root  515 Jul  5 19:17 meta.00
-rw-r--r-- 1 root root 2048 Jul  5 19:17 test_database.autogen.00004.00

# possible to backup specific retention policy or retention duration like follows

root@dlp:~# influxd backup -database test_database -retention one_day -since 2023-07-05T00:00:00Z /home/influxd_backup 
```

Restore a Backup database.

```sh
 # stop service before restoring

root@dlp:~# systemctl stop influxdb
# run restore command
# [-metadir] : target directory meta data is restored (follow is the default location)
# [-datadir] : target directory database is restored (follow is the default location)
# [-database] : target database name restored

root@dlp:~# influxd restore -metadir /var/lib/influxdb/meta -datadir /var/lib/influxdb/data -database test_database /home/influxd_backup

Using metastore snapshot: /home/influxd_backup/meta.00
2023/07/05 19:18:40 Restoring offline from backup /home/influxd_backup/test_database.*

root@dlp:~# chown -R influxdb:influxdb /var/lib/influxdb/{meta,data}

root@dlp:~# systemctl start influxdb
# verify resotored data

root@dlp:~# influx -username admin -password adminpassword -database test_database -precision rfc3339 -execute 'select * from weather'

name: weather
time                           location  temperature
----                           --------  -----------
2023-07-06T00:23:11.525199905Z hiroshima 20
2023-07-06T00:23:15.061760047Z hiroshima 22
2023-07-06T00:23:18.382129025Z osaka     18
2023-07-06T00:23:22.790775012Z osaka     19

```

# InfluxDB : Install Telegraf

 	
Install Telegraf which can collect or send various metrics.
Telegraf supports many Input /Output plugins, refer to the official site below.
⇒ https://docs.influxdata.com/telegraf/v1.27/plugins/

Install Telegraf. 

```sh
root@dlp:~# curl -fsSL https://repos.influxdata.com/influxdata-archive_compat.key -o /etc/apt/keyrings/influxdata-archive_compat.key

root@dlp:~# echo "deb [signed-by=/etc/apt/keyrings/influxdata-archive_compat.key] https://repos.influxdata.com/debian stable main" | tee /etc/apt/sources.list.d/influxdata.list

root@dlp:~# apt update

root@dlp:~# apt -y install telegraf 
```


Configure Telegraf.  
As an example, configure it to get CPU related metrics on the system and send them to InfluxDB.


```sh
 root@dlp:~# mv /etc/telegraf/telegraf.conf /etc/telegraf/telegraf.conf.org

root@dlp:~# vi /etc/telegraf/telegraf.conf
# create new

[global_tags]
  # set any tag if you need to add it on data
  dc = "hiroshima-01"

[agent]
  # collection interval for all inputs
  interval = "10s"
  # if set [true], then always collect on :00, :10, :20 ...
  round_interval = true
  # the size of writes that Telegraf sends to output plugins
  metric_batch_size = 1000
  # cost of higher maximum memory usage
  metric_buffer_limit = 10000
  # it is used to jitter the collection by a random amount
  collection_jitter = "0s"
  # flushing interval for all outputs
  flush_interval = "10s"
  # jitter the flush interval by a random amount
  # if flush_interval=10s, flush_jitter=5s, then flushes will happen every 10-15s
  flush_jitter = "0s"

[[outputs.influxdb]]
  # InfluxDB access URL
  url = "https://dlp.srv.world:8086"
  database = "telegraf"
  # if auth is enabled, set admin username and password
  username = "admin"
  password = "adminpassword"
  # if HTTPS is enabled, set the path to certificate
  # it needs [telegraf] user can read certificate
  tls_cert = "/etc/telegraf/server.crt"
  tls_key = "/etc/telegraf/server.key"
  # if using self signed certificate, set [true]
  insecure_skip_verify = true

# settings for CPU related metrics
[[inputs.cpu]]
  percpu = true
  totalcpu = true
  collect_cpu_time = false
  report_active = false

root@dlp:~# systemctl restart telegraf
# change retention policy
# follow means 30 days retention (default is infinite)

root@dlp:~# influx -username admin -password adminpassword -database telegraf -ssl -unsafeSsl \
-execute 'create retention policy "one_month" on "telegraf" duration 30d replication 1 default'
root@dlp:~# influx -username admin -password adminpassword -database telegraf -ssl -unsafeSsl \
-execute 'show retention policies'

name      duration shardGroupDuration replicaN default
----      -------- ------------------ -------- -------
autogen   0s       168h0m0s           1        false
one_month 720h0m0s 24h0m0s            1        true

# verify data

root@dlp:~# influx -username admin -password adminpassword -database telegraf -precision rfc3339 \
-execute 'select * from cpu' -ssl -unsafeSsl

name: cpu
time                 cpu       dc           host          usage_guest usage_guest_nice usage_idle        usage_iowait        usage_irq usage_nice usage_softirq usage_steal usage_system        usage_user
----                 ---       --           ----          ----------- ---------------- ----------        ------------        --------- ---------- ------------- ----------- ------------        ----------
2023-07-06T00:40:50Z cpu-total hiroshima-01 dlp.srv.world 0           0                99.84962406015165 0.05012531328320741 0         0          0             0           0.05012531328320741 0.05012531328320741
2023-07-06T00:40:50Z cpu0      hiroshima-01 dlp.srv.world 0           0                99.89969909729425 0                   0         0          0             0           0.10030090270812425 0
2023-07-06T00:40:50Z cpu1      hiroshima-01 dlp.srv.world 0           0                99.69969969970224 0.2002002002002003  0         0          0             0           0                   0.1001001001001046
2023-07-06T00:41:00Z cpu-total hiroshima-01 dlp.srv.world 0           0                99.6996996996977  0.25025025025024344 0         0          0             0           0.0500500500500478  0
2023-07-06T00:41:00Z cpu0      hiroshima-01 dlp.srv.world 0           0                100               0                   0         0          0             0           0                   0
2023-07-06T00:41:00Z cpu1      hiroshima-01 dlp.srv.world 0           0                99.49949949949769 0.40040040040039127 0         0          0             0           0.10010010010010004 0
.....
.....

```

For other System related metrics, configure like follows.

```sh
 root@dlp:~# vi /etc/telegraf/telegraf.conf
# add to the end

# memory
[[inputs.mem]]

# swap
[[inputs.swap]]

# disk
[[inputs.disk]]
  ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]

# disk IO
[[inputs.diskio]]

# kernel statistics (/proc/stat)
[[inputs.kernel]]

# processes
[[inputs.processes]]

# system (Uptime and so on)
[[inputs.system]]

# networking
[[inputs.net]]

root@dlp:~# systemctl restart telegraf
root@dlp:~# influx -username admin -password adminpassword -database telegraf -precision rfc3339 -execute 'show measurements' -ssl -unsafeSsl

name: measurements
name
----
cpu
disk
diskio
kernel
mem
net
processes
swap
system

root@dlp:~# influx -username admin -password adminpassword -database telegraf -precision rfc3339 \
-execute 'select * from disk' -ssl -unsafeSsl

name: disk
time                 dc           device free        fstype host          inodes_free inodes_total inodes_used label           mode path  total       used       used_percent
----                 --           ------ ----        ------ ----          ----------- ------------ ----------- -----           ---- ----  -----       ----       ------------
2023-07-06T00:42:10Z hiroshima-01 dm-0   26850648064 ext4   dlp.srv.world 1836871     1872304      35433       debian--vg-root rw   /     30006984704 1606123520 5.644081990323362
2023-07-06T00:42:10Z hiroshima-01 vda1   390417408   ext2   dlp.srv.world 124088      124440       352                         rw   /boot 476286976   60337152   13.385810672664078
2023-07-06T00:42:20Z hiroshima-01 dm-0   26850643968 ext4   dlp.srv.world 1836871     1872304      35433       debian--vg-root rw   /     30006984704 1606127616 5.644096384085451
2023-07-06T00:42:20Z hiroshima-01 vda1   390417408   ext2   dlp.srv.world 124088      124440       352                         rw   /boot 476286976   60337152   13.385810672664078
2023-07-06T00:42:30Z hiroshima-01 dm-0   26850627584 ext4   dlp.srv.world 1836871     1872304      35433       debian--vg-root rw   /     30006984704 1606144000 5.644153959133806
2023-07-06T00:42:30Z hiroshima-01 vda1   390417408   ext2   dlp.srv.world 124088      124440       352                         rw   /boot 476286976   60337152   13.385810672664078
2023-07-06T00:42:40Z hiroshima-01 dm-0   26850615296 ext4   dlp.srv.world 1836871     1872304      35433       debian--vg-root rw   /     30006984704 1606156288 5.644197140420074
2023-07-06T00:42:40Z hiroshima-01 vda1   390417408   ext2   dlp.srv.world 124088      124440       352                         rw   /boot 476286976   60337152   13.385810672664078
2023-07-06T00:42:50Z hiroshima-01 dm-0   26850611200 ext4   dlp.srv.world 1836871     1872304      35433       debian--vg-root rw   /     30006984704 1606160384 5.644211534182162
2023-07-06T00:42:50Z hiroshima-01 vda1   390417408   ext2   dlp.srv.world 124088      124440       352                         rw   /boot 476286976   60337152   13.385810672664078
.....
.....

root@dlp:~# influx -username admin -password adminpassword -database telegraf -precision rfc3339 \
-execute 'select * from diskio' -ssl -unsafeSsl

name: diskio
time                 dc           host          io_time iops_in_progress merged_reads merged_writes name read_bytes read_time reads weighted_io_time write_bytes write_time writes
----                 --           ----          ------- ---------------- ------------ ------------- ---- ---------- --------- ----- ---------------- ----------- ---------- ------
2023-07-06T00:42:10Z hiroshima-01 dlp.srv.world 12852   1                0            0             dm-0 216466432  1468      7177  10496            652558336   8772       14078
2023-07-06T00:42:10Z hiroshima-01 dlp.srv.world 12864   0                1765         6816          vda5 219669504  875       5543  12099            652558336   10967      7676
2023-07-06T00:42:10Z hiroshima-01 dlp.srv.world 20      0                0            0             dm-1 2351104    4         108   4                0           0          0
2023-07-06T00:42:10Z hiroshima-01 dlp.srv.world 60      0                31           0             vda1 4510720    26        181   37               1024        5          2
2023-07-06T00:42:10Z hiroshima-01 dlp.srv.world 8       0                0            0             vda2 2048       0         2     0                0           0          0
2023-07-06T00:42:10Z hiroshima-01 dlp.srv.world 12912   0                1796         6816          vda  226467840  907       5827  19087            652559360   10987      7683
2023-07-06T00:42:20Z hiroshima-01 dlp.srv.world 8       0                0            0             vda2 2048       0         2     0                0           0          0
2023-07-06T00:42:20Z hiroshima-01 dlp.srv.world 12944   0                1796         6833          vda  226467840  907       5827  19143            652682240   11015      7702
2023-07-06T00:42:20Z hiroshima-01 dlp.srv.world 20      0                0            0             dm-1 2351104    4         108   4                0           0          0
.....
.....

root@dlp:~# influx -username admin -password adminpassword -database telegraf -precision rfc3339 \
-execute 'select * from kernel' -ssl -unsafeSsl

name: kernel
time                 boot_time  context_switches dc           entropy_avail host          interrupts processes_forked
----                 ---------  ---------------- --           ------------- ----          ---------- ----------------
2023-07-06T00:42:10Z 1688600262 326271           hiroshima-01 256           dlp.srv.world 197483     2241
2023-07-06T00:42:20Z 1688600262 327887           hiroshima-01 256           dlp.srv.world 198368     2246
2023-07-06T00:42:30Z 1688600262 329862           hiroshima-01 256           dlp.srv.world 199985     2255
2023-07-06T00:42:40Z 1688600262 331479           hiroshima-01 256           dlp.srv.world 200899     2260
2023-07-06T00:42:50Z 1688600262 332989           hiroshima-01 256           dlp.srv.world 201781     2261
2023-07-06T00:43:00Z 1688600262 334920           hiroshima-01 256           dlp.srv.world 202809     2266
2023-07-06T00:43:10Z 1688600262 336435           hiroshima-01 256           dlp.srv.world 203543     2266
2023-07-06T00:43:20Z 1688600262 337797           hiroshima-01 256           dlp.srv.world 204260     2266
2023-07-06T00:43:30Z 1688600262 339376           hiroshima-01 256           dlp.srv.world 205095     2266
.....
.....

root@dlp:~# influx -username admin -password adminpassword -database telegraf -precision rfc3339 \
-execute 'select * from mem' -ssl -unsafeSsl

name: mem
time                 active    available  available_percent buffered cached    commit_limit committed_as dc           dirty  free       high_free high_total host          huge_page_size huge_pages_free huge_pages_total inactive  low_free low_total mapped    page_tables shared slab     sreclaimable sunreclaim swap_cached swap_free  swap_total total      used      used_percent       vmalloc_chunk vmalloc_total     vmalloc_used write_back write_back_tmp
----                 ------    ---------  ----------------- -------- ------    ------------ ------------ --           -----  ----       --------- ---------- ----          -------------- --------------- ---------------- --------  -------- --------- ------    ----------- ------ ----     ------------ ---------- ----------- ---------  ---------- -----      ----      ------------       ------------- -------------     ------------ ---------- --------------
2023-07-06T00:42:10Z 247738368 3720712192 90.6681505584557  21102592 597368832 3079430144   298844160    hiroshima-01 73728  3334131712 0         0          dlp.srv.world 2097152        0               0                416002048 0        0         133947392 1642496     589824 59518976 35606528     23912448   0           1027600384 1027600384 4103659520 151056384 3.6810164991465957 0             14073748835531776 19406848     0          0
2023-07-06T00:42:20Z 247754752 3708395520 90.36801181790052 21135360 602419200 3079430144   298844160    hiroshima-01 253952 3319271424 0         0          dlp.srv.world 2097152        0               0                418578432 0        0         135032832 1646592     589824 64823296 40636416     24186880   0           1027600384 1027600384 4103659520 160833536 3.9192709632986316 0             14073748835531776 19439616     0          0
2023-07-06T00:42:30Z 332931072 3719110656 90.6291235389821  21159936 602492928 3079430144   294649856    hiroshima-01 503808 3329859584 0         0          dlp.srv.world 2097152        0               0                329220096 0        0         134438912 1650688     589824 64827392 40640512     24186880   0           1027600384 1027600384 4103659520 150147072 3.6588579356603153 0             14073748835531776 19390464     0          0
2023-07-06T00:42:40Z 332947456 3713957888 90.50355834589318 21184512 602537984 3079430144   294649856    hiroshima-01 454656 3324661760 0         0          dlp.srv.world 2097152        0               0                331956224 0        0         135307264 1654784     589824 64864256 40677376     24186880   0           1027600384 1027600384 4103659520 155275264 3.7838242486550153 0             14073748835531776 19423232     0          0
2023-07-06T00:42:50Z 335060992 3709534208 90.39575992893289 21192704 602558464 3079430144   298844160    hiroshima-01 131072 3320209408 0         0          dlp.srv.world 2097152        0               0                332398592 0        0         135311360 1658880     589824 64864256 40677376     24186880   0           1027600384 1027600384 4103659520 159698944 3.8916226656152992 0             14073748835531776 19423232     0          0
2023-07-06T00:43:00Z 335077376 3709304832 90.39017038138681 21213184 602570752 3079430144   298844160    hiroshima-01 126976 3319951360 0         0          dlp.srv.world 2097152        0               0                334651392 0        0         135442432 1662976     589824 64864256 40677376     24186880   0           1027600384 1027600384 4103659520 159924224 3.8971123998123507 0             14073748835531776 19423232     0          0
2023-07-06T00:43:10Z 337203200 3709345792 90.39116851487718 21237760 602583040 3079430144   303038464    hiroshima-01 163840 3319951360 0         0          dlp.srv.world 2097152        0               0                334774272 0        0         135507968 1667072     589824 64864256 40677376     24186880   0           1027600384 1027600384 4103659520 159887360 3.896214079671015  0             14073748835531776 19423232     0          0
2023-07-06T00:43:20Z 339316736 3709370368 90.39176739497141 21254144 602591232 3079430144   303038464    hiroshima-01 20480  3319951360 0         0          dlp.srv.world 2097152        0               0                331501568 0        0         135573504 1667072     589824 64864256 40677376     24186880   0           1027600384 1027600384 4103659520 159862784 3.8956151995767914 0             14073748835531776 19423232     0          0
.....
.....

root@dlp:~# influx -username admin -password adminpassword -database telegraf -precision rfc3339 \
-execute 'select * from net' -ssl -unsafeSsl

name: net
time                 bytes_recv bytes_sent dc           drop_in drop_out err_in err_out host          icmp_inaddrmaskreps icmp_inaddrmasks icmp_incsumerrors icmp_indestunreachs icmp_inechoreps icmp_inechos icmp_inerrors icmp_inmsgs icmp_inparmprobs icmp_inredirects icmp_insrcquenchs icmp_intimeexcds icmp_intimestampreps icmp_intimestamps icmp_outaddrmaskreps icmp_outaddrmasks icmp_outdestunreachs icmp_outechoreps icmp_outechos icmp_outerrors icmp_outmsgs icmp_outparmprobs icmp_outredirects icmp_outsrcquenchs icmp_outtimeexcds icmp_outtimestampreps icmp_outtimestamps interface ip_defaultttl ip_forwarding ip_forwdatagrams ip_fragcreates ip_fragfails ip_fragoks ip_inaddrerrors ip_indelivers ip_indiscards ip_inhdrerrors ip_inreceives ip_inunknownprotos ip_outdiscards ip_outnoroutes ip_outrequests ip_reasmfails ip_reasmoks ip_reasmreqds ip_reasmtimeout packets_recv packets_sent tcp_activeopens tcp_attemptfails tcp_currestab tcp_estabresets tcp_incsumerrors tcp_inerrs tcp_insegs tcp_maxconn tcp_outrsts tcp_outsegs tcp_passiveopens tcp_retranssegs tcp_rtoalgorithm tcp_rtomax tcp_rtomin udp_ignoredmulti udp_incsumerrors udp_indatagrams udp_inerrors udp_memerrors udp_noports udp_outdatagrams udp_rcvbuferrors udp_sndbuferrors udplite_ignoredmulti udplite_incsumerrors udplite_indatagrams udplite_inerrors udplite_memerrors udplite_noports udplite_outdatagrams udplite_rcvbuferrors udplite_sndbuferrors
----                 ---------- ---------- --           ------- -------- ------ ------- ----          ------------------- ---------------- ----------------- ------------------- --------------- ------------ ------------- ----------- ---------------- ---------------- ----------------- ---------------- -------------------- ----------------- -------------------- ----------------- -------------------- ---------------- ------------- -------------- ------------ ----------------- ----------------- ------------------ ----------------- --------------------- ------------------ --------- ------------- ------------- ---------------- -------------- ------------ ---------- --------------- ------------- ------------- -------------- ------------- ------------------ -------------- -------------- -------------- ------------- ----------- ------------- --------------- ------------ ------------ --------------- ---------------- ------------- --------------- ---------------- ---------- ---------- ----------- ----------- ----------- ---------------- --------------- ---------------- ---------- ---------- ---------------- ---------------- --------------- ------------ ------------- ----------- ---------------- ---------------- ---------------- -------------------- -------------------- ------------------- ---------------- ----------------- --------------- -------------------- -------------------- --------------------
2023-07-06T00:42:10Z                       hiroshima-01                                 dlp.srv.world 0                   0                0                 0                   0               0            0             0           0                0                0                 0                0                    0                 0                    0                 0                    0                0             0              0            0                 0                 0                  0                 0                     0                  all       64            2             0                0              0            0          0               14113         0             0              14329         0                  0              1              10452          0             0           0             0                                         53              3                2             12              0                0          14831      -1          15          11178       36               0               1                120000     200        8                0                52              0            0             0           52               0                0                0                    0                    0                   0                0                 0               0                    0                    0
2023-07-06T00:42:10Z 62162348   710362     hiroshima-01 1934    0        0      0       dlp.srv.world                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        enp1s0                                                                                                                                                                                                                                                                                         16385        10271                                                                                                                                                                                                                                                                                                                                                                                                                  
2023-07-06T00:42:20Z                       hiroshima-01                                 dlp.srv.world 0                   0                0                 0                   0               0            0             0           0                0                0                 0                0                    0                 0                    0                 0                    0                0             0              0            0                 0                 0                  0                 0                     0                  all       64            2             0                0              0            0          0               14134         0             0              14350         0                  0              1              10473          0             0           0             0                                         54              3                2             13              0                0          14871      -1          16          11218       37               0               1                120000     200        8                0                52              0            0             0           52               0                0                0                    0                    0                   0                0                 0               0                    0                    0
2023-07-06T00:42:20Z 62162692   710446     hiroshima-01 1939    0        0      0       dlp.srv.world                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        enp1s0                                                                                                                                                                                                                                                                                         16392        10273                                                                                                                                                                                                                                                                                                                                                                                                                  
2023-07-06T00:42:30Z                       hiroshima-01                                 dlp.srv.world 0                   0                0                 0                   0               0            0             0           0                0                0                 0                0                    0                 0                    0                 0                    0                0             0              0            0                 0                 0                  0                 0                     0                  all       64            2             0                0              0            0          0               14173         0             0              14392         0                  0              1              10512          0             0           0             0                                         55              3                2             14              0                0          14908      -1          17          11255       38               0               1                120000     200        8                0                54              0            0             0           54               0                0                0                    0                    0                   0                0                 0               0                    0                    0
2023-07-06T00:42:30Z 62163866   710614     hiroshima-01 1944    0        0      0       dlp.srv.world                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        enp1s0                                                                                                                                                                                                                                                                                         16402        10275                                                                                                                                                                                                                                                                                                                                                                                                                  
2023-07-06T00:42:40Z                       hiroshima-01                                 dlp.srv.world 0                   0                0                 0                   0               0            0             0           0                0                0                 0                0                    0                 0                    0                 0                    0                0             0              0            0                 0                 0                  0                 0                     0                  all       64            2             0                0              0            0          0               14187         0             0              14407         0                  0              1              10526          0             0           0             0                                         56              3                2             15              0                0          14941      -1          18          11288       39               0               1                120000     200        8                0                54              0            0             0           54               0                0                0                    0                    0                   0                0                 0               0                    0                    0
2023-07-06T00:42:40Z 62164343   710614     hiroshima-01 1949    0        0      0       dlp.srv.world                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        enp1s0                                                                                                                                                                                                                                                                                         16408        10275                                                                                                                                                                                                                                                                                                                                                                                                                  
2023-07-06T00:42:50Z                       hiroshima-01                                 dlp.srv.world 0                   0                0                 0                   0               0            0             0           0                0                0                 0                0                    0                 0                    0                 0                    0                0             0              0            0                 0                 0                  0                 0                     0                  all       64            2             0                0              0            0          0               14208         0             0              14428         0                  0              1              10547          0             0           0             0                                         56              3                2             15              0                0          14962      -1          18          11309       39               0               1                120000     200        8                0                54              0            0             0           54               0                0                0                    0                    0                   0                0                 0               0                    0                    0
2023-07-06T00:42:50Z 62164673   710614     hiroshima-01 1954    0        0      0       dlp.srv.world                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        enp1s0                                                                                                                                                                                                                                                                                         16414        10275                                                                                                                                                                                                                                                                                                                                                                                                                  
.....
.....

root@dlp:~# influx -username admin -password adminpassword -database telegraf -precision rfc3339 \
-execute 'select * from processes' -ssl -unsafeSsl

name: processes
time                 blocked dc           dead host          idle paging running sleeping stopped total total_threads unknown zombies
----                 ------- --           ---- ----          ---- ------ ------- -------- ------- ----- ------------- ------- -------
2023-07-06T00:42:10Z 0       hiroshima-01 0    dlp.srv.world 50   0      0       55       0       105   121           0       0
2023-07-06T00:42:20Z 1       hiroshima-01 0    dlp.srv.world 50   0      0       54       0       105   121           0       0
2023-07-06T00:42:30Z 1       hiroshima-01 0    dlp.srv.world 50   0      0       54       0       105   120           0       0
2023-07-06T00:42:40Z 1       hiroshima-01 0    dlp.srv.world 50   0      0       54       0       105   120           0       0
2023-07-06T00:42:50Z 1       hiroshima-01 0    dlp.srv.world 50   0      0       54       0       105   121           0       0
2023-07-06T00:43:00Z 1       hiroshima-01 0    dlp.srv.world 50   0      0       54       0       105   121           0       0
2023-07-06T00:43:10Z 1       hiroshima-01 0    dlp.srv.world 50   0      0       54       0       105   121           0       0
2023-07-06T00:43:20Z 1       hiroshima-01 0    dlp.srv.world 50   0      0       54       0       105   121           0       0
.....
.....

root@dlp:~# influx -username admin -password adminpassword -database telegraf -precision rfc3339 \
-execute 'select * from swap' -ssl -unsafeSsl

name: swap
time                 dc           free       host          in out total      used used_percent
----                 --           ----       ----          -- --- -----      ---- ------------
2023-07-06T00:42:10Z hiroshima-01 1027600384 dlp.srv.world 0  0   1027600384 0    0
2023-07-06T00:42:20Z hiroshima-01 1027600384 dlp.srv.world 0  0   1027600384 0    0
2023-07-06T00:42:30Z hiroshima-01 1027600384 dlp.srv.world 0  0   1027600384 0    0
2023-07-06T00:42:40Z hiroshima-01 1027600384 dlp.srv.world 0  0   1027600384 0    0
2023-07-06T00:42:50Z hiroshima-01 1027600384 dlp.srv.world 0  0   1027600384 0    0
2023-07-06T00:43:00Z hiroshima-01 1027600384 dlp.srv.world 0  0   1027600384 0    0
2023-07-06T00:43:10Z hiroshima-01 1027600384 dlp.srv.world 0  0   1027600384 0    0
.....
.....

root@dlp:~# influx -username admin -password adminpassword -database telegraf -precision rfc3339 \
-execute 'select * from system' -ssl -unsafeSsl

name: system
time                 dc           host          load1 load15 load5 n_cpus n_unique_users n_users uptime uptime_format
----                 --           ----          ----- ------ ----- ------ -------------- ------- ------ -------------
2023-07-06T00:42:10Z hiroshima-01 dlp.srv.world 0     0      0     2      1              1       3868    1:04
2023-07-06T00:42:20Z hiroshima-01 dlp.srv.world 0     0      0     2      1              1       3878    1:04
2023-07-06T00:42:30Z hiroshima-01 dlp.srv.world 0     0      0     2      1              1       3888    1:04
2023-07-06T00:42:40Z hiroshima-01 dlp.srv.world 0     0      0     2      1              1       3898    1:04
2023-07-06T00:42:50Z hiroshima-01 dlp.srv.world 0     0      0     2      1              1       3908    1:05
2023-07-06T00:43:00Z hiroshima-01 dlp.srv.world 0     0      0     2      1              1       3918    1:05
2023-07-06T00:43:10Z hiroshima-01 dlp.srv.world 0     0      0     2      1              1       3928    1:05
.....
.....

```