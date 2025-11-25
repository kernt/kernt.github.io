Apache Kafka is a distributed streaming platform. It is useful for building real-time streaming data pipelines to get data between the systems or applications. Another useful feature is real-time streaming applications that can transform streams of data or react on a stream of data.

For more info refer to [https://kafka.apache.org/](https://kafka.apache.org/). There is no other documentation in the world that explains it better!

This tutorial will help you to install Apache Kafka on Debian distributions and finally run Apache Kafka as a service on the system. Keep calm and follow step by step!

Step 1: Download Apache Kafka from official downloads page using wget and extract it into a folder in /usr/local

```sh
 wget [http://mirrors.estointernet.in/apache/kafka/2.3.0/kafka_2.12-2.3.0.tgz](http://mirrors.estointernet.in/apache/kafka/2.3.0/kafka_2.12-2.3.0.tgz)  
 tar xzf kafka_2.12-2.3.0.tgz  
 mv kafka_2.12-2.3.0 /usr/local/kafka
```

Step 2: Change the configuration of Kafka server config and uncomment the line as shown below:

```sh
cd /usr/local/kafka  
nano config/server.properties# see kafka.server.KafkaConfig for additional details and defaults############################# Server Basics ############################## The id of the broker. This must be set to a unique integer for each broker.  
broker.id=0############################# Socket Server Settings ############################## The address the socket server listens on. It will get the value returned from   
# java.net.InetAddress.getCanonicalHostName() if not configured.  
#   FORMAT:  
#     listeners = listener_name://host_name:port  
#   EXAMPLE:  
#     listeners = PLAINTEXT://your.host.name:9092  
**listeners=PLAINTEXT://:9092 --> uncomment and configure kafka port**
```

Step 3: Starting Zookeeper as a Service

```sh
nano /etc/systemd/system/zookeeper.service**Edit the file like:**[Unit]  
Requires=network.target remote-fs.target  
After=network.target remote-fs.target[Service]  
Type=simple  
User=root  
ExecStart=/usr/local/kafka/bin/zookeeper-server-start.sh /usr/local/kafka/config/zookeeper.properties  
ExecStop=/usr/local/kafka/bin/zookeeper-server-stop.sh  
Restart=on-abnormal[Install]  
WantedBy=multi-user.target**Save, enable the service and finally run the service**> systemctl enable zookeeper.service  
service zookeeper start
```

Step 4: Starting Kafka as a service

```
nano /etc/systemd/system/kafka.service**Edit the file like:**[Unit]  
Requires=zookeeper.service  
After=zookeeper.service[Service]  
Type=simple  
User=root  
ExecStart=/bin/sh -c '/usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties > /usr/local/kafka/kafka.log 2>&1'  
ExecStop=/usr/local/kafka/bin/kafka-server-stop.sh  
Restart=on-abnormal[Install]  
WantedBy=multi-user.target**Save, enable the service and finally run the service**> systemctl enable kafka.service  
service kafka start
```

Step 5: Check if Kafka Service is working

```
**Create a Topic**

bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic test-topic

**Start Publisher over the Topic**
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test-topic

**Start Consumer over the Topic in another terminal**
> sudo bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic test-topic
```

On successful setup you will be able to run Kafka as a service under Pub-Sub mode, where whenever you publish messages over test-topic, you will be able to see them over consumer console.