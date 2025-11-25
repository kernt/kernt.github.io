---
tags:
  - nosql
  - datenbanken
  - redix
---
Redis is an open-source, in-memory data structure store widely used as a database, cache, and message broker. Its high performance and flexibility make it a popular choice in various tech sectors. 

* iredis  - CLI for Redis with auto-completion and syntax highlighting

## Key Features of Redis

- **Fast Data Access**: Redis stores data in memory, resulting in quick retrieval. This makes it ideal for caching applications.
- **Multiple Data Structures**: Unlike traditional tables-based databases, Redis supports various data structures like strings, hashes, lists, and sets.
- **Real-Time Messaging**: Redis offers Publish/Subscribe messaging patterns, enabling real-time application communication.
- **Data Persistence**: While primarily an in-memory database, Redis allows for periodic data saving on disk, offering a balance between speed and durability.
- **Scalability and Availability**: Features like replication, sharding, and Redis Sentinel ensure high availability and scalability across multiple nodes.

## Use-Cases für Redis

- **Caching**: Redis is commonly used to cache frequently accessed data, speeding up retrieval times.
- **Session Storage**: It’s well-suited for storing user session data, especially in high-traffic web applications.
- **Message Queuing**: The pub/sub capabilities make Redis an excellent choice for message queuing systems.

Understanding Redis’s capabilities can significantly enhance your technology stack. The upcoming guide will detail how to install Redis on Debian 12 Bookworm, Debian 11 Bullseye, or Debian 10 Buster. We’ll cover two installation methods: one using Debian’s default repository and another using the official Redis repository for the latest version. Both methods will use Command Line Interface (CLI) commands. Stay tuned for the step-by-step instructions.

## Pre-installation Steps with Redis Installation

### Update Debian System Packages

Before installing Redis or any other software, the initial step is to ensure your system’s packages are up-to-date. This critical preliminary step helps mitigate potential conflicts during the subsequent installation process. It ensures that all system dependencies are current, minimizing potential issues arising from outdated software packages.

In a Debian system, you can do this by running the following command in the terminal:

```bash
sudo apt update && sudo apt upgrade
```

### Install Required Packages for Redis Installation

Once the Debian system packages are up-to-date, you’ll need to install specific software packages necessary for the Redis installation.

You can install these packages by running the following command:

```bash
sudo apt install software-properties-common apt-transport-https curl ca-certificates -y
```

Let’s dissect this command to understand it better:

- `sudo apt install`: This command is used to install packages on your Debian system.
- `software-properties-common`: This package provides specific scripts for securely managing software and source code.
- `apt-transport-https`: This package is necessary for the apt package manager to retrieve packages over the ‘https’ protocol.
- `curl`: This is a command-line tool for transferring data using various network protocols. It helps download files, among other things.
- `ca-certificates`: This package is necessary to verify the security of websites. It provides a set of Certifying Authority (CA) certificates.
- `-y`: This option automatically answers ‘yes’ to any prompts during installation.

# Auswählen einer Redis Installation Methoden

## Option 1: Install Redis with APT Debian Repository

While Debian includes Redis in its default software offering, the version may not be the most recent. Debian favors stability and generally provides security or significant updates only. This may make the Redis version on Debian older but arguably more stable. This might be the perfect route if your intended usage of Redis doesn’t demand the most recent features.

To install Redis via this method, you’ll need to execute the following command in your terminal:

```bash
sudo apt install redis
```

The `sudo` command provides superuser privileges, while `apt install redis` installs the Redis software package onto your Debian system.

## Option 2: Install Redis with Redis.io APT Repository

The second method might be more appealing for those requiring or desiring the most up-to-date version of Redis. It involves importing the APT repository from the official Redis.io repository. This repository regularly receives bug fixes, security patches, and feature updates.

### Import the Redis.io Repository

First, import the GPG key. GPG, or GNU Privacy Guard, helps with secure communication and data storage. The GPG key verifies the data source and ensures no one altered it during download.

Use this command to import the GPG key:

```bash
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
```

Next, we import the repository with the following command:

```bash
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
```

### Update Packages Index After Redis.io APT Import

With the new repository added, you’ll need to update your system’s package index to know the packages available from the new source. Do this by running the following command:

```bash
sudo apt-get update
```

### Install Redis via APT Command

With everything set up, you can install Redis from the Redis.io repository. If you already had Redis installed from the Debian repository, you may see an upgrade instead. The following command installs Redis, along with the Redis server and tools:

```bash
sudo apt install redis redis-server redis-tools
```

### Verify Redis Installation

After installing, it’s always a good idea to verify the installation. You can do this with the `apt-cache policy` command, confirming that you have installed the Redis.io version. Here’s how to use it:

```bash
apt-cache policy redis
```

Use the following command to activate the Redis instance and set it to start during system boot:

```bash
sudo systemctl enable redis-server --now
```

To verify that Redis is running without errors, use this command:

```bash
systemctl status redis-server
```

This command provides information about the `redis-server` service, which should now be running.

## Inspecting Redis: Post-installation Verifications on Debian

### Listening Ports

By default, Redis listens on localhost using port 6379. To ensure that Redis is indeed actively listening as expected, use the following command:

```bash
ps -ef | grep redis
```

This command displays running processes involving Redis, which should reflect the listening status of Redis on port 6379.

### Ping Testing the Redis Service

A ping test is one of the simplest ways to verify that your Redis service is operational. This will establish a connection to your Redis service, verifying that it’s up, running, and ready to respond to commands.

First, connect to the Redis service with the following command:

```bash
redis-cli
```

After running this command, your terminal should display `127.0.0.1:6379`, indicating that you’re connected to Redis on the localhost. Now, you can ping the Redis service as follows:

```bash
ping
```

### Exiting the Redis Instance

After you complete the verification checks and confirm Redis works correctly, exit the Redis instance by typing:

```bash
exit
```

We have successfully installed Redis on a Debian system, either using the Debian APT repository or the official Redis.io APT repository.

# Konfiguration und Setup Redis

Redis stands out because of its flexibility and ability to fit various needs. This section guides you on adjusting Redis for caching and network access and adding a password for increased security.

### Accessing the Redis Configuration File

The journey of tailoring Redis begins with the Redis configuration file located at `/etc/redis/redis.conf`. You’ll need to open this file using a text editor like `nano`.

Run the following command to do:

```bash
sudo nano /etc/redis/redis.conf
```

### Designating Maximum Memory for Caching

A use case for Redis is caching data to boost application performance. To designate a specific quantity of memory for this purpose in Redis, append the following lines after the configuration file:

```bash
maxmemory 500mb 
maxmemory-policy allkeys-lru
```

In this scenario, we’ve allocated 500 MB of memory for Redis. Feel free to adjust this quantity based on your server’s specifications and your application’s requirements.

Once the assigned memory reaches capacity, Redis will remove keys following the Least Recently Used (LRU) policy.

## Setting up Network Access

By default, Redis listens only on the localhost interface. You can change this setting to allow Redis to listen on all interfaces or specific IP addresses/subnets.

First, find line 69 in the configuration file.

### Option 1: Listening to All Network Interfaces

To have Redis listen on all network interfaces, you can comment out the `bind` line by adding a `#` at the start:

```bash
# bind 127.0.0.1 ::1
```

### Option 2: Binding to a Specific IP Address or Subnet

To bind Redis to a specific IP address or subnet, replace the `bind` line with your preferred IP or subnet:

```bash
bind 0.0.0.0/0
```

or

```bash
bind 192.150.5.0/24
```

When binding Redis to an IP address or subnet, it is strongly recommended that a password be enforced for heightened security.

## Enforcing Password Authentication

To add an extra layer of protection to your Redis instance, you can set a password for authentication.

Search for the line starting with `# requirepass` (around line 507), uncomment it, and set a strong password:

```bash
requirepass YourStrongPasswordHere
```

Replace `YourStrongPasswordHere` with a sturdy password that includes a mix of uppercase and lowercase letters, numbers, and special symbols.

## Committing Changes and Restarting Redis

After you’ve made the necessary adjustments to the configuration file, save your changes by pressing `Ctrl + O`, and then exit the `nano` editor by pressing `Ctrl + X`.

Finally, restart the Redis service to enact the new settings:

```bash
sudo systemctl restart redis
```

This command requests the system to halt the Redis service and immediately start it up again, thus ensuring that Redis operates under the new configuration parameters you’ve set.

To ensure that the Redis service was restarted successfully and is running correctly, you can check the status of the service by executing the following command:

```bash
sudo systemctl status redis
```

This command will return information about the Redis service, including its status (active or inactive), uptime, and log messages you may have seen earlier when confirming the installation.

## Configure a Firewall for Redis with UFW

Securing your Redis server requires setting up a robust firewall. Redis doesn’t include a firewall by default. Also, Debian doesn’t offer an out-of-the-box firewall solution unless you’re proficient with iptables. But there’s a straightforward method you can use, even if you’re a beginner. This method involves the Uncomplicated Firewall (UFW), a user-friendly firewall tool.

### Ensuring UFW is Installed

First, check if your Debian server has UFW. If it doesn’t, use the following command to install it:

```bash
sudo apt install ufw -y
```

In this command, `sudo` is used to execute the command with root privileges, `apt install` is the command to install a package, `ufw` is the package to be installed, and `-y` is an option that automatically answers yes to any prompts, thereby allowing the installation to proceed without further user input.

### Enabling UFW

Once UFW is installed, it must be enabled to start functioning and protecting your system. You can enable UFW with this command:

```bash
sudo ufw enable
```

This command tells UFW to activate and apply any rules you have set or will set.

### Configuring UFW Rules for Redis

Now that UFW is activated, you can configure it to protect your Redis server. You might need to set up different firewall rules depending on your specific situation and whether your Redis installation is a standalone server or part of a cluster.

#### For an Additional Network IP Server Instance

If you have a standalone Redis server that you want to protect with UFW, use this command:

```bash
sudo ufw allow proto tcp from <ip address> to any port 6379
```

Here, you replace `<ip address>` with the IP address of the server from which you want to allow connections. This command tells UFW to allow TCP connections from that IP address to any system on port 6379, the default port for Redis.

#### For a Cluster Network with Multiple Instances

If you are working with a cluster of Redis servers, you can allow connections from an entire subnet. To do this, use the following command:

```bash
sudo ufw allow proto tcp from <ip address>/24 to any port 6379
```

In this command, replace `<ip address>` with the IP address of the subnet from which you want to allow connections. Note that this command implies significant trust in all the devices on this subnet, as it opens up the Redis server to all of them. Therefore, only use this command if you are sure that your internal network is secure.

### Verifying the UFW Configuration

After configuring your UFW rules, confirming that they work as intended is essential. Just as you pinged your Redis server at the beginning of this guide to ensure it was operational, you can also test your new firewall rules. You can do this using the `redis-cli` command:

```bash
redis-cli -h  <ip address> ping
```

Again, replace `<ip address>` with the IP address of your Redis server. If your firewall rules have been set up correctly and the Redis server is operating as it should, the output from this command should be a simple `pong`.

In this command, `redis-cli` is the command-line interface for Redis, `-h` specifies the host (your server’s IP address), and `ping` is a command to check connectivity to the Redis server.

### Interpreting the Output

The output from the `redis-cli` command will help you confirm if the firewall rules are working as expected.

```bash
pong
```

This output is a good sign. A `pong` response indicates that the Redis server is accessible and ready to accept commands. It confirms that the firewall rules you’ve set up in UFW work correctly, allowing the server to communicate as intended.

## Additional Redis Configuration Examples

As discussed earlier, Redis is a flexible in-memory data store with many features. It can be used for caching and messaging. Changing its configuration file allows you to make Redis work how you want. Here, we will show more configuration options for Redis.

### Setting the Key-Value Expiration Policy

To set a default time-to-live (TTL) for keys in Redis, change the maxmemory-policy and maxmemory-samples settings in the configuration file. Use the following command to open the file:

```bash
sudo nano /etc/redis/redis.conf
```

Look for the `maxmemory-policy` setting and configure it to your liking. For instance, if you wish to establish a policy that expires keys according to the Least Recently Used (LRU) algorithm, update the configuration as such:

```bash
maxmemory-policy volatile-lru
```

Next, locate the `maxmemory-samples` setting. This will dictate the number of samples that Redis should review to arrive at the eviction decision:

```bash
maxmemory-samples 5
```

### Enabling TCP Keepalive

Activating TCP keepalive can aid in detecting and closing idle connections, which in turn releases valuable system resources. Locate the `tcp-keepalive` setting in the `/etc/redis/redis.conf` file to enable it:

```bash
tcp-keepalive 300
```

In this example, we set the keepalive interval to 300 seconds. You should adjust this figure according to your server’s requirements and conditions.

### Setting up Slow Log Monitoring

Slow logs provide a valuable mechanism for pinpointing performance issues. They do so by logging queries that exceed a specified duration. Find the `slowlog-log-slower-than` and `slowlog-max-len` settings in the `/etc/redis/redis.conf` file to configure slow logs:

```bash
slowlog-log-slower-than 10000
slowlog-max-len 128
```

The above example sets the slowlog threshold to 10,000 microseconds (equivalent to 10 milliseconds). The slow log will maintain the 128 most recent entries. You should adjust these values to match your specific requirements and conditions.

### Enabling Redis Event Notifications

Redis can create notifications for specific events, providing a powerful tool for monitoring and debugging. To activate event notifications, locate the `notify-keyspace-events` setting in the `/etc/redis/redis.conf` file:

```bash
notify-keyspace-events ExA
```

Here, we configure Redis to dispatch notifications for expired and evicted keys. The official Redis documentation is a comprehensive resource for more information on event notifications and available choices.

### Adjusting the Redis Logging Level

To better troubleshoot and monitor Redis, change its logging level. You can do this in the /etc/redis/redis.conf file by adjusting the loglevel setting:

```bash
loglevel notice
```

We’ve set the default logging level to ‘notice’. Other options are ‘debug’, ‘verbose’, ‘notice’, and ‘warning’. Choose the level that best suits your needs.

```bash
sudo systemctl restart redis-server
```

This command restarts Redis, and the changes will be active.

# Redis mit Docker

docker-compose.yaml

```yaml
services:  
  redis-node-1:  
    image: redis:7.2.3  
    command: ["/tmp/redis.sh", "10.11.12.13", "7001"]  
    volumes:  
      - ./redis.sh:/tmp/redis.sh  
  redis-node-2:  
    image: redis:7.2.3  
    command: ["/tmp/redis.sh", "10.11.12.13", "7002"]  
    volumes:  
      - ./redis.sh:/tmp/redis.sh  
  redis-node-3:  
    image: redis:7.2.3  
    command: ["/tmp/redis.sh", "10.11.12.13", "7003"]  
    volumes:  
      - ./redis.sh:/tmp/redis.sh  
  redis-node-4:  
    image: redis:7.2.3  
    command: ["/tmp/redis.sh", "10.11.12.13", "7004"]  
    volumes:  
      - ./redis.sh:/tmp/redis.sh  
  redis-node-5:  
    image: redis:7.2.3  
    command: ["/tmp/redis.sh", "10.11.12.13", "7005"]  
    volumes:  
      - ./redis.sh:/tmp/redis.sh  
  redis-node-6:  
    image: redis:7.2.3  
    command: ["/tmp/redis.sh", "10.11.12.13", "7006"]  
    volumes:  
      - ./redis.sh:/tmp/redis.sh  
  redis-cluster-creator:  
    image: redis:7.2.3  
    command: redis-cli -a 'SUPER_SECRET_PASSWORD' --cluster create 10.11.12.13:7001 10.11.12.13:7002 10.11.12.13:7003 10.11.12.13:7004 10.11.12.13:7005 10.11.12.13:7006 --cluster-replicas 1 --cluster-yes  
    depends_on:  
      - redis-node-1  
      - redis-node-2  
      - redis-node-3  
      - redis-node-4  
      - redis-node-5  
      - redis-node-6  
      - redis-proxy  
  redis-proxy:  
    image: haproxytech/haproxy-alpine:2.4  
    volumes:  
      - ./haproxy:/usr/local/etc/haproxy:ro  
    ports:  
      - 8404:8404  
      - 7001-7006:9001-9006  
      - 7101-7106:9101-9106  
    depends_on:  
      - redis-node-1  
      - redis-node-2  
      - redis-node-3  
      - redis-node-4  
      - redis-node-5  
      - redis-node-6  
  redis-insight:  
    container_name: redis-insight  
    image: redislabs/redisinsight:1.14.0  
    ports:  
      - 8001:8001  
    depends_on:  
      - redis-proxy
```

We have;

- 6 server nodes
- 1 cluster creater node. Status of this container should be “stopped” after it created cluster successfully. (if you wish you can exec into any container and run cluster command instead of this container).
- TCP proxy
- UI (redis insight)

_10.11.12.13_ is IP of local machine(actually IP of TCP Proxy). You should change it (may be to your PCName)

# redis.sh

Instead of creating separate redis.conf file for each node, I created a sh file which generates redis.conf file according to given parameters then starts server with this file. This is more flexible for a demo.

```sh
ANNOUNCE_IP=$1  
ANNOUNCE_PORT=$(expr $2)  
ANNOUNCE_BUS_PORT=$(expr $ANNOUNCE_PORT + 100)  
  
CONF_FILE="/tmp/redis.conf"  
```
  
# generate redis.conf file

```sh
echo "port 6379  
cluster-enabled yes  
cluster-config-file nodes.conf  
cluster-node-timeout 5000  
appendonly yes  
loglevel debug  
requirepass SUPER_SECRET_PASSWORD  
masterauth  SUPER_SECRET_PASSWORD  
protected-mode no  
cluster-announce-ip $ANNOUNCE_IP  
cluster-announce-port $ANNOUNCE_PORT  
cluster-announce-bus-port $ANNOUNCE_BUS_PORT  
" >> $CONF_FILE  
```
  
# start server  
redis-server $CONF_FILE

Each redis server running on port 6379 but their cluster announce IP and ports are different.

> cluster-announce-port is used for client-to-server communications.
> 
> cluster-announce-bus-port is used for server-to-server communications.

# haproxy.cfg

```
global  
  stats socket /var/run/api.sock user haproxy group haproxy mode 660 level admin expose-fd listeners  
  log stdout format raw local0 info  
  
defaults  
  mode tcp  
  timeout client 600s  
  timeout connect 5s  
  timeout server 600s  
  timeout http-request 10s  
  log global  
  
frontend stats  
  mode http  
  bind *:8404  
  stats enable  
  stats uri /stats  
  stats refresh 10s  
  stats admin if LOCALHOST  
```
  
# frontend 

```sh
frontend redisfe  
  bind :9001-9006  
  bind :9101-9106  
  use_backend redisbe1 if { dst_port 9001 }  
  use_backend redisbe2 if { dst_port 9002 }  
  use_backend redisbe3 if { dst_port 9003 }  
  use_backend redisbe4 if { dst_port 9004 }  
  use_backend redisbe5 if { dst_port 9005 }  
  use_backend redisbe6 if { dst_port 9006 }  
  use_backend redisbusbe1 if { dst_port 9101 }  
  use_backend redisbusbe2 if { dst_port 9102 }  
  use_backend redisbusbe3 if { dst_port 9103 }  
  use_backend redisbusbe4 if { dst_port 9104 }  
  use_backend redisbusbe5 if { dst_port 9105 }  
  use_backend redisbusbe6 if { dst_port 9106 }  
```
  
# Server 1

```sh
backend redisbe1  
  server be1 redis-node-1:6379 check  
  
backend redisbusbe1  
  server busbe1 redis-node-1:16379 check  
```
  
# Server 2

```sh
backend redisbe2  
  server be2 redis-node-2:6379 check  
  
backend redisbusbe2  
  server busbe2 redis-node-2:16379 check  
```
  
# Server 3

```ah
backend redisbe3
  server be3 redis-node-3:6379 check
  
backend redisbusbe3
  server busbe3 redis-node-3:16379 check
```
  
# Server 4

```sh
backend redisbe4  
  server be4 redis-node-4:6379 check  
  
backend redisbusbe4  
  server busbe4 redis-node-4:16379 check  
```
  
# Server 5

```sh
backend redisbe5  
  server be5 redis-node-5:6379 check  
  
backend redisbusbe5  
  server busbe5 redis-node-5:16379 check  
```
  
# Server 6

```sh
backend redisbe6  
  server be6 redis-node-6:6379 check  
  
backend redisbusbe6  
  server busbe6 redis-node-6:16379 check
```

HAProxy listens 9001–9006 ports for client access (client-to-server), 9101–9106 port for server access(server-to-server). HAProxy stats can be accessible from [http://localhost:8404/stats](http://localhost:8404/stats)

Access order:

- client => haproxy-container:700x => haproxy:900x => redis-node-x:6379
- redis-node-* => haproxy-container:710x => haproxy:910x => redis-node-x:16379

# Redis Insight

Redis insight is a really good UI for redis. For this setup you can access it from [http://localhost:8001/](http://localhost:8001/)

# Basic Operation for Database

 This is the Basic Usage of Redis with [redis-cli] client program.
Following examples are basic one, you can see more commands on Official Site below.
⇒ https://redis.io/commands
This is the basic Usage of Keys. 

```sh
 root@dlp:~# redis-cli

127.0.0.1:6379> auth password 
OK

# set Value of a Key
127.0.0.1:6379> set key01 value01 
OK

# get Value of a Key
127.0.0.1:6379> get key01 
"value01"

# delete a Key
127.0.0.1:6379> del key01 
(integer) 1

# determine if a Key exists or not (1 means true)
127.0.0.1:6379> exists key01 
(integer) 0

# set Value of a Key only when the Key does not exist yet
# integer 0 means Value is not set because the Key already exists
127.0.0.1:6379> setnx key01 value02 
(integer) 1

# set Value of a Key with expiration date (60 means Value will expire after 60 second)
127.0.0.1:6379> setex key01 60 value01 
OK

# set expiration date to existing Key
127.0.0.1:6379> expire key01 30 
(integer) 1

# add Values to a Key
127.0.0.1:6379> append key01 value02 
(integer) 15

# get substring of Value of a Key : [Key] [Start] [End]
127.0.0.1:6379> substr key01 0 3 
"valu"

127.0.0.1:6379> set key02 1 
OK

# increment integer Value of a Key 
127.0.0.1:6379> incr key02 
(integer) 2

# increment integer Value of a Key by specified value
127.0.0.1:6379> incrby key02 100 
(integer) 102

# decrement integer Value of a Key
127.0.0.1:6379> decr key02 
(integer) 101

# decrement integer Value of a Key by specified value
127.0.0.1:6379> decrby key02 51 
(integer) 50

# set values of some Keys
127.0.0.1:6379> mset key01 value01 key02 value02 key03 value03 
OK

# get some Values of Keys 
127.0.0.1:6379> mget key01 key02 key03 
1) "value01"
2) "value02"
3) "value03"

# rename existing Key
127.0.0.1:6379> rename key03 key04 
OK
127.0.0.1:6379> mget key01 key02 key03 key04 
1) "value01"
2) "value02"
3) (nil)
4) "value03"

# rename existing Key but if renamed Key already exists, command is not run
127.0.0.1:6379> renamenx key01 key02 
(integer) 0
127.0.0.1:6379> mget key01 key02 key03 key04 
1) "value01"
2) "value02"
3) (nil)
4) "value03"

# get number of Keys on current Database
127.0.0.1:6379> dbsize 
(integer) 3

# move a key to another Database
127.0.0.1:6379> move key04 1 
(integer) 1
127.0.0.1:6379> select 1 
OK
127.0.0.1:6379[1]> get key04 
"value03"

# delete all Keys on current Database
127.0.0.1:6379> flushdb 
OK

# delete all Keys on all Database
127.0.0.1:6379> flushall 
OK
127.0.0.1:6379> quit 

# process with reading data from stdout
root@dlp:~# echo 'test_words' | redis-cli -a password --no-auth-warning -x set key01 
OK
root@dlp:~# redis-cli -a password --no-auth-warning get key01 
"test_words\n"

```

It's possible to use CAS operation (Check And Set) with watch command on Redis.  
If another process changed value of the Key between multi - exec, change is not applied to the Key.

```sh
# watch a key
127.0.0.1:6379> watch key01 
OK
127.0.0.1:6379> get key01 
"test_words\n"
127.0.0.1:6379> multi 
OK
127.0.0.1:6379> set key01 value02 
QUEUED
127.0.0.1:6379> exec 
1) OK
```

https://medium.com/@ithesadson/setting-up-redisinsight-ui-on-kubernetes-with-nginx-ingress-and-basic-authentication-a4b749b49085

https://medium.com/@ozanbozkurtt96/redis-3-node-cluster-with-redis-sentinel-high-availability-and-failover-setup-guide-2-db8758ac2e25

