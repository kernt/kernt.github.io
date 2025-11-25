n today’s world, data handling requires not just speed but also reliability, durability, and high availability. In this blog, we’ll explore the key concepts of **replication**, **sharding**, and how MongoDB combines these mechanisms to achieve both performance and durability. We’ll also look into how a cluster is configured to ensure fault tolerance, and walk through the practical aspects of setting up a MongoDB cluster manually.

# Why Do We Need Multiple Database Servers?

As data volume grows, we encounter two primary challenges: **performance** and **durability**. A single database server may not suffice to handle high traffic or ensure data is always available, especially during system failures. To address this, we use **replication** and **sharding** within a MongoDB cluster.

- **Durability** ensures data is not lost even in the event of hardware failure.
- **Performance** helps in distributing workload across multiple servers, allowing faster data access

## Replication and Replicas

**Replication** in MongoDB is the process of copying data across multiple servers, which helps in both durability and fault tolerance. A MongoDB cluster typically consists of multiple nodes (servers), where one node is designated as the **primary node** (also known as the master), and the others are **secondary nodes** (slaves).

- **Primary Node**: This is where all the write operations take place.
- **Secondary Nodes**: These replicate data from the primary node and can be used for read operations.

If the primary node fails, MongoDB initiates an **election** process among the remaining nodes to choose a new primary. This ensures that the system remains operational without any manual intervention.

## High Availability and Fault Tolerance

With replication, MongoDB achieves high availability. Since data is replicated across multiple nodes, the system can tolerate failures. If one node crashes, another can take over. This architecture allows the system to maintain both **availability** and **data durability**.

# The Role of Sharding in MongoDB

While replication handles durability, **sharding** improves performance by distributing data across multiple nodes. Sharding allows MongoDB to horizontally scale large datasets by splitting them across various servers.

## How Sharding Works

In MongoDB, **sharding** involves distributing data based on a **shard key**, which could be any attribute such as name, age, or phone number. The key defines how data will be partitioned across different nodes.

For example:

- Node 1 stores ages between 1–30.
- Node 2 stores ages between 31–60.
- Node 3 stores ages between 61–90.

The shard key is chosen based on the most frequently queried attribute. For instance, if users mostly search by age, then age would be the shard key.

```
{  
  name: "Eric",  
  age: 30  
}
```

In this example, if **age** is the shard key, MongoDB will store the document in the node that handles the age range of 30. This makes read operations faster as each query is directed to the relevant shard based on the key.

## Combining Sharding and Replication

Sharding alone improves read performance, but it doesn’t guarantee availability or durability. To address this, MongoDB combines **sharding** with **replication**. Each shard in a cluster has its own replica set to ensure that data is not lost and remains available, even if one shard fails.

For example, let’s say we have a node (Node B) that stores data for ages 31–60. Behind Node B, two more nodes, Node M and Node N, replicate its data to ensure durability. If Node B fails, one of the secondary nodes (M or N) can take over.

This combination of **sharding** and **replication** allows MongoDB to handle both performance and data durability at scale.

# MongoDB Cluster Architecture

![](https://miro.medium.com/v2/resize:fit:624/0*EapaB_A0PlUSJwzb.png)

## 1. MongoDB Router (mongos)

The **MongoDB Router (mongos)** acts as an entry point or intermediary between the client and the shards in the cluster. It doesn’t store any data but plays a critical role in directing queries and write operations to the correct shard based on the shard key.

Without mongos, the client would need to know where each piece of data is stored, which would significantly complicate database operations. Mongos handles this complexity and allows clients to interact with the cluster seamlessly.

## 2. Metadata or Config Servers

The **Config Servers** (metadata servers) store all the metadata about the cluster’s architecture, including the distribution of data across shards. They store information about how data is partitioned and which shard holds which part of the data.

The config server cluster is essential because, without it, mongos wouldn’t be able to route the data correctly, and the system wouldn’t know how data is distributed across the shards.

## 3. Shard Servers

**Shards** are the actual database servers that store the data. Each shard is a replica set that contains a primary node and secondary nodes. Sharding divides the data into smaller, manageable pieces, and each shard holds a portion of the data.

**Why are Shard Servers needed?**

- **Data Distribution**: Sharding distributes data across multiple nodes, allowing for horizontal scaling. This means the database can grow by adding more machines rather than increasing the size of a single machine.
- **Fault Tolerance and Durability**: Since each shard is a replica set, the primary node handles writes, and secondary nodes replicate the data. If the primary node fails, an election process automatically promotes one of the secondary nodes to primary.
- **Scalability**: Sharding allows the cluster to handle large amounts of data by dividing it into smaller chunks, which can be stored across multiple servers. This ensures that even massive datasets can be managed efficiently.

## Putting It All Together: MongoDB Cluster Workflow

When a client sends a request to MongoDB:

1. **mongos** receives the request.
2. **mongos** checks with the **config server** to identify the shard(s) containing the requested data.
3. It then routes the request to the relevant **shard server** for processing.

In the case of a write operation, mongos sends the data to the **primary node** of the appropriate shard. The secondary nodes replicate the data to ensure fault tolerance and durability.

# Practical Setup of a MongoDB Cluster

Now that we understand the components of MongoDB architecture, let’s go through the steps of setting up a MongoDB cluster using **Docker on AWS EC2 Instance.**

## Step-by-Step MongoDB Cluster Setup:

![](https://miro.medium.com/v2/resize:fit:700/0*9L02qhHC2roBBG70.jpg)

## Step1: Install Docker on Amazon Linux.

The first step is to install the docker so that we can start creating our cluster.

`yum install docker -y`

![](https://miro.medium.com/v2/resize:fit:700/1*VSC1f0xjIf0PqWvTZI4xmQ.png)

## Step2: Installing Mongodb tools.

- For installing `mongodb-org` you first need to add the yum repository.
- Create a `/etc/yum.repos.d/mongodb-org-7.0.repo` file so that you can install MongoDB directly using `yum`:

```
[mongodb-org-7.0]  
name=MongoDB Repository  
baseurl=https://repo.mongodb.org/yum/amazon/2023/mongodb-org/7.0/x86_64/  
gpgcheck=1  
enabled=1  
gpgkey=https://pgp.mongodb.com/server-7.0.asc
```

![](https://miro.medium.com/v2/resize:fit:700/1*fxKLm1wPSjPSPK1N3dtR2A.png)

- To install the latest stable version of MongoDB, run the following command:

`sudo yum install -y mongodb-org`

![](https://miro.medium.com/v2/resize:fit:700/1*BGEtK7TXKKcfZdGh_gnYTQ.png)

- Confirm by running `mongosh` command:

![](https://miro.medium.com/v2/resize:fit:700/1*jpy5OFOHp9g2SinVGgM8vw.png)

## Step3: Installing Docker-Compose.

- Next step is to install docker-compose because we will launch our shards servers or config servers individually in a group.

```
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose  
sudo chmod +x /usr/local/bin/docker-compose  
docker-compose version
```

![](https://miro.medium.com/v2/resize:fit:700/1*h0ysfCrHrwI8bmezMnIFHA.png)

## Step4: Launching Config servers.

- The first step in launching config servers is to create docker compose file `configsvr.yaml` where we describe name of each container and image used and command that will be run inside the container.

  
```yaml
version: '3'  
  
services:  
  
  cfgsvr1:  
    container_name: cfgsvr1  
    image: mongo  
    command: mongod --configsvr --replSet cfgrs --port 27017 --dbpath /data/db  
    ports:  
      - 40001:27017  
    volumes:  
      - cfgsvr1:/data/db  
  
  cfgsvr2:  
    container_name: cfgsvr2  
    image: mongo  
    command: mongod --configsvr --replSet cfgrs --port 27017 --dbpath /data/db  
    ports:  
      - 40002:27017  
    volumes:  
      - cfgsvr2:/data/db  
  
  cfgsvr3:  
    container_name: cfgsvr3  
    image: mongo  
    command: mongod --configsvr --replSet cfgrs --port 27017 --dbpath /data/db  
    ports:  
      - 40003:27017  
    volumes:  
      - cfgsvr3:/data/db  
  
volumes:  
  cfgsvr1: {}  
  cfgsvr2: {}  
  cfgsvr3: {}
```

- The command is used to configure them as a replica set to ensure high availability.
- These containers are exposed with port numbers `**40001, 40002, 40003**` .
- **Launch config servers :**

`docker-compose -f configsvr.yaml up -d`

![](https://miro.medium.com/v2/resize:fit:700/1*jmtYJZLlLRIKTzX-eCOuLA.png)

![](https://miro.medium.com/v2/resize:fit:700/1*Fw73G1XnqWiaRoVvkOv8xQ.png)

- Let’s try to connect to one of the server using `mongosh` command.

`mongosh <system-ip>:40001`

![](https://miro.medium.com/v2/resize:fit:700/1*knEwzEJSNDeXv6SbdcHyCA.png)

- Although it is successfully connecting but when we run a command, it throws error.
- This is because these nodes did not know about each other, so even after configuring them as a replica, it will not work because they cannot contest election and without primary and secondary classifications, nothing will work here.

**Initiate replica set:**

```
rs.initiate(  
  {  
    _id: "cfgrs",  
    configsvr: true,  
    members: [  
      { _id : 0, host : "172.31.35.127:40001" },  
      { _id : 1, host : "172.31.35.127:40002" },  
      { _id : 2, host : "172.31.35.127:40003" }  
    ]  
  }  
)
```

![](https://miro.medium.com/v2/resize:fit:700/1*RW_EdZAcriSv11wXyykSOg.png)

`rs.status()`

- This command will tell you everything about the replica set that how many replicas has been added, is election contested or not and which node will become primary and which will become secondary. Mostly that node where you perfrom these steps will become primary.

## Step5: Set Up the Shard Servers.

- Again we need to create a docker compose file `shard1svr.yaml` . Each shard server will also be part of its own replica set to ensure durability.

  
```yaml
version: '3'  
  
services:  
  
  shard1svr1:  
    container_name: shard1svr1  
    image: mongo  
    command: mongod --shardsvr --replSet shard1rs --port 27017 --dbpath /data/db  
    ports:  
      - 50001:27017  
    volumes:  
      - shard1svr1:/data/db  
  
  shard1svr2:  
    container_name: shard1svr2  
    image: mongo  
    command: mongod --shardsvr --replSet shard1rs --port 27017 --dbpath /data/db  
    ports:  
      - 50002:27017  
    volumes:  
      - shard1svr2:/data/db  
  
  shard1svr3:  
    container_name: shard1svr3  
    image: mongo  
    command: mongod --shardsvr --replSet shard1rs --port 27017 --dbpath /data/db  
    ports:  
      - 50003:27017  
    volumes:  
      - shard1svr3:/data/db  
  
volumes:  
  shard1svr1: {}  
  shard1svr2: {}  
  shard1svr3: {}
```

- **Start config servers:**

`docker-compose -f shard1svr.yaml up -d`

![](https://miro.medium.com/v2/resize:fit:700/1*wCYML54FRtKiyFXgFsLmLw.png)

- Again we need to follow the same steps and initiate the replica set by going inside one of the shard server.

`mongosh <system-ip>:50001`

![](https://miro.medium.com/v2/resize:fit:700/1*WyIOWSphnZ8HBWkq1GXlgg.png)

- Let’s check the status with `rs.status()` command.

![](https://miro.medium.com/v2/resize:fit:589/1*lVkxe_ckJiyTavAi0YA4-g.png)

![](https://miro.medium.com/v2/resize:fit:700/1*mssONarJlRLIgTslhghWTQ.png)

- Now this is the first shard cluster, we also need to create one more shard cluster with the same steps.
- The docker compose file and the necessary commands are:

  
```yaml
version: '3'  
  
services:  
  
  shard2svr1:  
    container_name: shard2svr1  
    image: mongo  
    command: mongod --shardsvr --replSet shard2rs --port 27017 --dbpath /data/db  
    ports:  
      - 50004:27017  
    volumes:  
      - shard2svr1:/data/db  
  
  shard2svr2:  
    container_name: shard2svr2  
    image: mongo  
    command: mongod --shardsvr --replSet shard2rs --port 27017 --dbpath /data/db  
    ports:  
      - 50005:27017  
    volumes:  
      - shard2svr2:/data/db  
  
  shard2svr3:  
    container_name: shard2svr3  
    image: mongo  
    command: mongod --shardsvr --replSet shard2rs --port 27017 --dbpath /data/db  
    ports:  
      - 50006:27017  
    volumes:  
      - shard2svr3:/data/db  
  
volumes:  
  shard2svr1: {}  
  shard2svr2: {}  
  shard2svr3: {}
```

`docker-compose -f shard2svr.yaml up -d`

`mongosh <system-ip>:50004`

```
rs.initiate(  
  {  
    _id: "shard2rs",  
    members: [  
      { _id : 0, host : "172.31.35.127:50004" },  
      { _id : 1, host : "172.31.35.127:50005" },  
      { _id : 2, host : "172.31.35.127:50006" }  
    ]  
  }  
)  
  
rs.status()

```
## Step6 : Launch the MongoDB Router (mongos)

The next step is to set up the **mongos** router. This router will direct client requests to the appropriate shard.

- Creating a docker compose file for mongos server.

  
```yaml
version: '3'  
  
services:  
  
  mongos:  
    container_name: mongos  
    image: mongo  
    command: mongos --configdb cfgrs/172.31.35.127:40001,172.31.35.127:40002,172.31.35.127:40003 --bind_ip 0.0.0.0 --port 27017  
    ports:  
      - 60000:27017
```

In this command:

- `mongos` is the router service.
- The `--configdb` flag specifies the config server replica set and its server address.
- **Launch the mongos server.**

`docker-compose -f mongos.yaml up -d`

![](https://miro.medium.com/v2/resize:fit:700/1*lJ2AsYGAIx1oqFyjv5ReHg.png)

- Connect to the mongos server.

![](https://miro.medium.com/v2/resize:fit:700/1*gV4scEJOIq276zC5-U3IEQ.png)

`sh.status()`

This command is used to check the shards status.

![](https://miro.medium.com/v2/resize:fit:700/1*EmALXKzEKOosCpsMVQOC8Q.png)

- And currently there are no shards available and for this we need to add the shards.

sh.addShard("shard1rs/172.31.35.127:50001,172.31.35.127:50002,172.31.35.127:50003")

![](https://miro.medium.com/v2/resize:fit:700/1*3vfgg2LlaB8dXLb3sJlTvA.png)

![](https://miro.medium.com/v2/resize:fit:700/1*NCHMuArZ5dyos579WyD3CA.png)

- Now one shard is added in shards column.
- Similarly we will add second shard cluster also.

![](https://miro.medium.com/v2/resize:fit:700/1*5FEPuhwEvgKUQgzsN0j_Hw.png)

![](https://miro.medium.com/v2/resize:fit:700/1*_1NTMJZGzrkb9JB-XpMuNQ.png)

## Step 5: Test the Setup:

- Enable sharding in the database `userdb` .

sh.enableSharding("userdb")

![](https://miro.medium.com/v2/resize:fit:700/1*hwWcxZR7jl8-P29ooWdM-g.png)

- Now after enable sharding in database, we also need to enable sharding in collection otherwise data will not be distributed accordingly. For this we need to create collection with below command.

sh.shardCollection("userdb.persons", {"age": "hashed"})

- Here `userdb` is our database and `persons` is our collection and `age` is our shard key and `hashed` is a algorithm used to store data accordingly in different shards.

![](https://miro.medium.com/v2/resize:fit:700/1*mzISsQ-7rQBzm4OYyhE-cw.png)

- Let’s see the shard distribution of our collection `persons` that how much data is stored in each shard.

`db.persons.getShardDistribution()`

![](https://miro.medium.com/v2/resize:fit:700/1*NPKgYup0ihN051Ms9sBIqg.png)

- On storing data, you will see changes here which will tell you that where the data will get stored.

`db.persons.insert({name: "Harsh", age: 20})`

![](https://miro.medium.com/v2/resize:fit:700/1*F0-lSrdLHee2u9fO_HOAoQ.png)

![](https://miro.medium.com/v2/resize:fit:700/1*HmcsK7dW_LilbwgaXXA53Q.png)

`db.persons.insert({name: "Jack", age: 11})`

![](https://miro.medium.com/v2/resize:fit:700/1*bqqhnU_uARuobm1EWVN4BQ.png)

![](https://miro.medium.com/v2/resize:fit:700/1*MpqEhaXDlJc25_aVktZ8_Q.png)

- From the values, we come to know that both the documents stored in shard2.
- Let’s insert one more document.

`db.persons.insert({name: "Jack", age: 5})`

![](https://miro.medium.com/v2/resize:fit:700/1*stDSnysqezdPctCFlEgxgg.png)

![](https://miro.medium.com/v2/resize:fit:700/1*z1pdEMH423nzMAGfaV-ayQ.png)

- Here this data stored in shard1 and these decision is taken based on shard key `age` .

# Conclusion

MongoDB’s cluster architecture — comprising the **mongos router**, **config servers**, and **shard servers** — is designed to achieve high performance, availability, and fault tolerance at scale. By distributing data across multiple shards and replicating data across nodes, MongoDB ensures that the system is both scalable and durable.

The combination of **sharding** for performance and **replication** for fault tolerance makes MongoDB a powerful solution for handling large-scale, mission-critical applications.