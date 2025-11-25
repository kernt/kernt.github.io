---
tags:
  - cluster
  - keepalived
  - ipvsadm
---
**High Availability** (HA) refers to systems that are designed to be reliable, operational, and available for use over long periods without any downtime or with minimal downtime. The goal is to ensure that services remain available and operational to the highest extent possible. Here are some key aspects of High Availability:

1. Fault Tolerance:  
    High Availability systems can withstand failures without experiencing downtime. This is often achieved through redundancy, where critical system components are duplicated to ensure that a failure in one does not result in system downtime.
2. Failover:  
    In the event of a failure, traffic or operations are automatically rerouted to standby or backup systems to maintain service continuity.
3. Recovery:  
    High Availability systems have mechanisms for rapid recovery from failures to restore normal operations.
4. Maintenance:  
    These systems are designed to allow for maintenance, updates, and repairs without disrupting service availability.
5. Monitoring and Detection:  
    Continuous monitoring to detect and respond to failures promptly.
6. Load Balancing:  
    Distributing workloads across multiple servers or systems to ensure that no single point becomes a bottleneck, contributing to high availability.
7. Scalability:  
    The ability to increase capacity by connecting multiple hardware or software entities to work as a single logical unit.

High Availability can be measured using metrics like uptime, which is often expressed as a percentage, with higher values like 99.9% (“three nines”) or 99.999% (“five nines”) indicating more reliable and available systems.

By incorporating these principles, organizations can ensure that their services remain reliable and available to users, which is crucial for maintaining trust and meeting service level agreements (SLAs).

**Load Balancing** refers to the process of distributing network traffic across several servers to ensure that no single server becomes a bottleneck. This improves responsiveness and availability of applications and websites. Here are some key aspects of Load Balancing:

1. Distribution of Traffic:  
    Load Balancing helps in efficiently distributing incoming network traffic across a group of servers, ensuring that no single server becomes overwhelmed with too much traffic. This is crucial for maintaining high performance and reliability.
2. Redundancy and Reliability:  
    By spreading the load, this process helps in mitigating the risk of server failures. If one server fails, the traffic is redirected to the remaining operational servers without any downtime.
3. Scalability:  
    As traffic grows, more servers can be added to the load balancing pool to handle the increased load, making load balancing a key component for scaling applications.
4. Performance Optimization:  
    Load Balancers can distribute traffic based on various factors, such as server performance or the type of content being served, ensuring that resources are utilized efficiently.
5. Session Persistence:  
    In scenarios where session persistence is crucial, Load Balancers can be configured to direct all requests from a particular client to the same server for as long as the session remains active.
6. Health Checks:  
    Load Balancers continuously monitor the health of servers, ensuring that traffic is not sent to servers that are unavailable or underperforming.
7. Algorithm-Based Distribution:  
    Load Balancers use various algorithms like Round Robin, Least Connections, or IP Hash to decide which server should handle a particular request.
8. Security:  
    They can also provide additional security by hiding the structure of the network and the servers within from the outside world.

Load Balancing is a fundamental concept in ensuring that networks and applications can handle varying loads while providing a seamless user experience. It’s a crucial part of modern IT infrastructure, particularly for high traffic websites and critical business applications.

**Linux Virtual Server** (LVS) is a scalable and high-performance server built on a Linux system. It’s often used for load balancing TCP/IP traffic among multiple servers. One of its features is TCP state synchronization, which is essential for maintaining a consistent and reliable service, especially in a high availability environment.

Here’s a breakdown of LVS TCP state synchronization:

1. State Synchronization:  
    State synchronization allows multiple load balancers to share information about the current state of TCP connections. This is crucial for ensuring that all load balancers have a consistent view of the network, enabling them to make intelligent routing decisions.
2. Failover Capabilities:  
    In a scenario where one of the load balancers fails, TCP state synchronization enables another load balancer to take over without losing existing connections. This failover capability is a key aspect of maintaining high availability and ensuring uninterrupted service.
3. Connection Persistence:  
    By synchronizing the state of TCP connections, LVS ensures that client connections are not dropped or lost during failovers. Clients can continue to interact with the application without experiencing disruptions.
4. Clustered Environment:  
    In a clustered environment, having synchronized TCP states among all nodes ensures that traffic is evenly distributed and that no single node becomes a bottleneck. This also allows for more effective load distribution and better utilization of resources.
5. Real-time Synchronization:  
    The synchronization process occurs in real time or near real time, ensuring that all load balancers have the most current information about the state of TCP connections.
6. Reduced Downtime:  
    By ensuring that other load balancers can seamlessly take over in the event of a failure, TCP state synchronization significantly reduces the potential downtime and enhances the reliability of the system.

LVS TCP state synchronization is an indispensable feature for environments requiring high availability and seamless failover capabilities. By maintaining a synchronized state of TCP connections among load balancers, LVS ensures that the system can effectively handle client requests, even in the face of individual node failures, thereby promoting a reliable and consistent user experience.

To ensure uninterrupted operation of your production services, let’s proceed by integrating the aforementioned three elements of High Availability (HA), Load Balancing (LB), and Linux Virtual Server (LVS) to work together.

To enhance the scalability and distribution of our HA/LB LVS, we will forego the standard master-backup solution and instead adopt a dual master and backup approach.

To emulate this setup and also to keep it simple for learning purposes, we will utilize 2 HA/LB and backend servers, also referred to as Real Servers (RS). If you wish to scale your HA/LB layer later, it will be easy to add more dual master and backup configurations as needed.

Let’s first illustrate our objectives on the following diagram:

![](gHDd5uQHVuuyGqkyHcGnUw.webp)

As depicted in the diagram, you can observe that the connection between clients and Real Servers passes through the LB/HA nodes, but only in a one-way direction since the Real Servers respond to clients directly bypassing the HA/LB nodes. _To enable a functional setup, it’s essential to configure the Keepalived VRRP Virtual IP addresses (VIP) on the Real Servers as well,_ **_along with setting the ARP notification and response levels for all Real Servers (in our case RS1 through RS2), restricted within the network_**_._ We will explore this further in an example later on_._

> Now, let’s proceed with the Keepalived and ipvsadm LVS configurations.

Let’s assume we have the following nodes:

```
- **HA/LB nodes:**  
    **ha_lb_01**  
    - 172.16.0.10/24 static IP  
    - 10.0.0.10/24 VRRP VIP floating IP  
    **ha_lb_02**  
    - 172.16.0.11/24 static IP  
    - 10.0.0.11/24 VRRP VIP floating IP
- **Backend/Real Servers:  
    rs1  
    **- 172.16.0.100/24  
    **rs2**  
    - 172.16.0.101/24
```


By inserting the line `lvs_sync_daemon $IFACE $VRRP_NAME id $ID_Number` in the `global_defs` section of the Keepalived configuration, Keepalived can directly manage LVS with ipvsadm.

```sh
global_defs {  
    lvs_sync_daemon $IFACE $VRRP_NAME id $ID_Number   
    ...  
    ...  
    ...  
    }
```

However, in this scenario, we would end up with a single master and backup, which doesn’t align with our objective of having a dual master and backup setup.  
Therefore, we will bypass the addition of `lvs_sync_daemon` in the `global_defs` section of Keepalived; instead, we will configure that part using custom notify scripts.

- **In ha_lb_01:  
    **`apt -y install keepalived`  
    `cd /etc/keepalived`  
    `vi keepalived.conf` — use your preferred editor

```c
global_defs {  
    router_id ha_lb_lvs # Replace the part on the right with your preferred name.  
    enable_script_security  
    script_user root  
    }  
  
vrrp_instance VI_1 { # vrrp protocol  
    state MASTER # MASTER server of lvs  
    interface ens18 # Replace with your interface  
    virtual_router_id 50 # Virtual routing  
    priority 100 # The masters should be assigned the highest priority.  
    advert_int 1 # VRRP Advert interval in seconds  
    authentication { # Verification (This authentication section is optional)  
    auth_type PASS    
    auth_pass 1111 # Replace this with your preferred one.  
    }  
    virtual_ipaddress {   
        10.0.0.10/24  
    }  
    notify "/etc/keepalived/notify.sh"  
}  
  
vrrp_instance VI_2 {  
    state BACKUP  
    interface ens18 # Replace with your interface  
    virtual_router_id 51  
    priority 90  
    advert_int 1  
    authentication { # Verification (This authentication section is optional)  
        auth_type PASS  
        auth_pass 2222 # Replace this with your preferred one.  
    }  
    virtual_ipaddress {  
        10.0.0.11/24  
    }  
    notify "/etc/keepalived/notify.sh"  
}  
  
virtual_server 10.0.0.10 80 {  
    delay_loop 6 # Real Server Check Interval  
    lb_algo lc # Scheduling: Least-Connection  
    lb_kind DR # The forwarding type: DR - Director Routing  
    protocol TCP # Service protocol  
    real_server 172.16.0.100 80 { # Backend/Real Server address  
        weight 1  
        HTTP_GET { # Application layer detection  
            url {  
              path / # Define the URL to monitor  
              status_code 200 # The response code used to determine that the above detection mechanism is in a healthy state  
            }  
            connect_timeout 3 # Timeout duration of connection request  
            retry 2 # retry count  
            delay_before_retry 2 # Delay before retrying  
        }  
    }  
    real_server 172.16.0.101 80 {  
        weight 2  
        HTTP_GET {  
            url {  
                path /  
                status_code 200  
            }  
            connect_timeout 3  
            retry 2  
            delay_before_retry 2  
        }  
    }  
}  
  
virtual_server 10.0.0.11 80 {  
    delay_loop 6  
    lb_algo lc  
    lb_kind DR  
    protocol TCP  
    real_server 172.16.0.100 80 {  
        weight 1  
        HTTP_GET {  
            url {  
              path /  
              status_code 200  
            }  
            connect_timeout 3  
            retry 2  
            delay_before_retry 2  
        }  
    }  
    real_server 172.16.0.101 80 {  
        weight 2  
        HTTP_GET {  
            url {  
                path /  
                status_code 200  
            }  
            connect_timeout 3  
            retry 2  
            delay_before_retry 2  
        }  
    }  
}
```


- The notify script on `**ha_lb_01**`  
    `notify.sh`

```sh
#!/bin/bash  
           
TYPE=$1  
NAME=$2  
ENDSTATE=$3  
IFACE=ens18 # Replace with your interface  
  
case $ENDSTATE in  
    "BACKUP") # Perform action for transition to BACKUP state  
       ipvsadm --start-daemon backup --syncid 2 --mcast-interface=$IFACE  
              exit 0  
              ;;  
    "FAULT")  # Perform action for transition to FAULT state  
              exit 0  
              ;;  
    "STOP")  # Perform action for transition to STOP state  
       ipvsadm --stop-daemon master  
       ipvsadm --stop-daemon backup  
              exit 0  
              ;;  
    "MASTER") # Perform action for transition to MASTER state  
       ipvsadm --start-daemon master --syncid 1 --mcast-interface=$IFACE  
              exit 0  
              ;;  
    *)        echo "Unknown state ${ENDSTATE}"  
              exit 1  
              ;;  
esac
```

- `service keepalived restart`
- **In ha_lb_02:  
    **`apt -y install keepalived`  
    `cd /etc/keepalived`  
    `vi keepalived.conf` — use your preferred editor

```c
global_defs {
    router_id ha_lb_lvs
    enable_script_security
    script_user root
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens18 # Replace with your interface
    virtual_router_id 50
    priority 90
    advert_int 1
    authentication { # Verification (This authentication section is optional)
        auth_type PASS
        auth_pass 1111 # Replace this with your preferred one.
    }
    virtual_ipaddress {
       10.0.0.10/24
    }
    notify "/etc/keepalived/notify.sh"
}

vrrp_instance VI_2 {
    state MASTER
    interface ens18 # Replace with your interface
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication { # Verification (This authentication section is optional)
        auth_type PASS
        auth_pass 2222 # Replace this with your preferred one.
    }
    virtual_ipaddress {
        10.0.0.11/24
    }
    notify "/etc/keepalived/notify.sh"
}

virtual_server 10.0.0.10 80 {
    delay_loop 6 
    lb_algo lc 
    lb_kind DR 
    protocol TCP 
    real_server 172.16.0.100 80 { 
        weight 1
        HTTP_GET { 
            url {
              path / 
              status_code 200 
            }
            connect_timeout 3 
            retry 2 
            delay_before_retry 2 
        }
    }
    real_server 172.16.0.101 80 {
        weight 2
        HTTP_GET {
            url {
                path /
                status_code 200
            }
            connect_timeout 3
            retry 2
            delay_before_retry 2
        }
    }
}

virtual_server 10.0.0.11 80 {
    delay_loop 6
    lb_algo lc
    lb_kind DR
    protocol TCP
    real_server 172.16.0.100 80 {
        weight 1
        HTTP_GET {
            url {
              path /
              status_code 200
            }
            connect_timeout 3
            retry 2
            delay_before_retry 2
        }
    }
    real_server 172.16.0.101 80 {
        weight 2
        HTTP_GET {
            url {
                path /
                status_code 200
            }
            connect_timeout 3
            retry 2
            delay_before_retry 2
        }
    }
}
}
```

- The notify script on `**ha_lb_02**`  
    `notify.sh`

```sh
#!/bin/bash  
  
TYPE=$1  
NAME=$2  
ENDSTATE=$3  
IFACE=ens18 # Replace with your interface  
  
case $ENDSTATE in  
    "BACKUP") # Perform action for transition to BACKUP state  
       ipvsadm --start-daemon backup --syncid 1 --mcast-interface=$IFACE  
              exit 0  
              ;;  
    "FAULT")  # Perform action for transition to FAULT state  
              exit 0  
              ;;  
    "STOP")  # Perform action for transition to STOP state  
       ipvsadm --stop-daemon master  
       ipvsadm --stop-daemon backup  
              exit 0  
              ;;  
    "MASTER") # Perform action for transition to MASTER state  
       ipvsadm --start-daemon master --syncid 2 --mcast-interface=$IFACE  
              exit 0  
              ;;  
    *)        echo "Unknown state ${ENDSTATE}"  
              exit 1  
              ;;  
esac
```

- `service keepalived restart`

You may notice in the notify scripts that the ID of `syncid` is different in 2 servers to prevent conflicts between the two masters, ensuring that one master serves as a backup for the other and vice versa.  
If everything is configured correctly, executing the command `ipvsadm -l --daemon` will yield the following output:

- **ha_lb_01**

```sh
root@ha_lb_01:/etc/keepalived# ipvsadm -l --daemon                                         
master sync daemon (mcast=ens18, syncid=1, maxlen=1472, group=224.0.0.81, port=8848, ttl=1)  
backup sync daemon (mcast=ens18, syncid=2, maxlen=1472, group=224.0.0.81, port=8848, ttl=1)
```

- **ha_lb_02**

```sh
root@ha_lb_02:/etc/keepalived# ipvsadm -l --daemon                                          
master sync daemon (mcast=ens18, syncid=2, maxlen=1472, group=224.0.0.81, port=8848, ttl=1)   
backup sync daemon (mcast=ens18, syncid=1, maxlen=1472, group=224.0.0.81, port=8848, ttl=1) 
```

Now is the time to prepare our backend/Real Servers to operate in a unidirectional manner. As mentioned earlier, the connection initiates from the clients to the HA/LB nodes, which then forwards it to the Backend/Real Server. The Real Server responds directly to the clients, bypassing the HA/LB nodes:

As I mentioned earlier `_it's essential to configure the Keepalived VRRP Virtual IP addresses (VIP) on the Real Servers as well, along with setting the ARP notification and response levels for all Real Servers (in our case RS1 through RS2), restricted within the network. We will explore this further in an example later on._`

- Wee need to setup the following on both `**rs1**`, **172.16.0.100 and** `**rs2**` **, 172.16.0.101**

```sh
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore  
echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore  
echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce  
echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
```

```sh
ip a a 10.0.0.10/32 dev lo # or # ip a a 10.0.0.10/32 dev ens18 # Replace ens18 with your interface  
ip a a 10.0.0.11/32 dev lo # or # ip a a 10.0.0.10/32 dev ens18 # Replace ens18 with your interface
```

And voila, everything is ready for the stateful High Availability and Load Balancing to operate.

From here on, scaling the HA/LB nodes will be straightforward. I’ll only share an image of the scenario to provide context and insight for further scale of the HA/LB layer, should the need arise.

![](Avx8fh0rWMXSfWbpARHicg.webp)

Short Reference Guide for ipvsadm Command Line:

- IPVS connection entries:  
    `ipvsadm -L -n -c`
- List the virtual server table  
    `ipvsadm -L -n`
- Daemon information output  
    `ipvsadm -l --daemon`

For more information, refer to the man page for ipvsadm by executing `man ipvsadm`.