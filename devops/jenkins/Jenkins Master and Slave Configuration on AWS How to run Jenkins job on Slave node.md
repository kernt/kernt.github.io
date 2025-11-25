## **Jenkins Master**

The Master’s job is to handle:

- Scheduling build jobs.
- Dispatching builds to the slaves for the actual execution.
- Monitor the slaves (possibly taking them online and offline as required).
- Recording and presenting the build results.
- A Master instance of Jenkins can also execute build jobs directly.

# Jenkins Slave

A Slave is a Java executable that runs on a remote machine. The following are the characteristics of Jenkins's Slaves:

- It hears requests from the Jenkins Master instance.
- Slaves can run on a variety of operating systems.
- The job of a Slave is to do as they are told to, which involves executing build jobs dispatched by the Master.
- You can configure a project to always run on a particular Slave machine or a particular type of Slave machine, or simply let Jenkins pick the next available Slave.

## Prerequisites:

- AWS Account
- Jenkins Master Server on AWS EC2
- Slave server on AWS EC2
- Familiarity with Git and Docker
- Master and slave should have the same Java and Maven version

## Steps to Follow:

- Jenkins Master server and Slave node on AWS
- Install Java on the Slave node
- Configure slave node under manage nodes of Jenkins Master
- Download agent.jar onto the slave node
- Run the java command to connect with the Jenkins server

## Step 1: Install Java on Slave Node

Before installing Java on the Slave node let’s first check the version of Java on the Jenkins Master server:

Before installing Java on the Slave node let’s first check the version of Java on the Jenkins Master server:

```
[root@jenkins-server ~]# java --version  
openjdk 11.0.18 2023-01-17 LTS  
OpenJDK Runtime Environment (Red_Hat-11.0.18.0.10-1.amzn2.0.1) (build 11.0.18+10-LTS)  
OpenJDK 64-Bit Server VM (Red_Hat-11.0.18.0.10-1.amzn2.0.1) (build 11.0.18+10-LTS, mixed mode, sharing)
```

The command to install Java on Amazon Linux 2 EC2 machine is :

`amazon-linux-extras install java-openjdk11`

![](wPlu03RqL66yjsK_ihTYOA.webp)

## Step 2: Configure slave node under manage nodes of Jenkins Master

Go to Manage Jenkins > Manage Nodes and Clouds:

![](jJ_RmIF3kl8J2SE62LTxGg.webp)

On the next screen click on **New Node**:

![](MRcyrnn8C7vIKyCvSCo-Pw.webp)

Name your new node, add it as a **Permanent Agent,** and click on the **Create** button:

![](sDAe_gPYJ1evL7VNGMHNsA.webp)

The following screen is important as we have to provide some important settings. The name of your node will appear as provided in the previous screen, provide some description about your node and then mention the **Number of executors** which means the maximum number of concurrent builds that Jenkins may perform on this node. In our case, we will mention

2.

![](MGjhxOakwNdRvSIiYnIO4A.webp)

On the other part of this screen, you need to mention the **Remote root directory** on the slave node on which all the jobs will be saved. After that provide the **label** which is used to group multiple agents into one logical group. Under Usage select **Use this node as much as possible** which is the default setting and Jenkins uses this node freely, under Launch method select **Launch agent by connecting it to the controller** which Allows an agent to be connected to the Jenkins controller whenever it is ready.

![](oDpRMfldRFP_8Fi72DOicg.webp)

The final part gives the option to **Disable WorkDir** which contains logs, we will keep that unchecked as we want to store the logs in our workspace and finally check the **Use WebSocket** checkbox which Use WebSocket to connect to the Jenkins master rather than the TCP port. Click on **Save** to proceed.

![](3JRy8XsE7eU0KLur70cUDQ.webp)

At this point, our newly added slave node will be showing offline:

![](he317YKEb99n1_byCK4z7ww.webp)

## Step 3:Download agent.jar onto the slave node

Let’s click on our slave node and there we should see two options to activate our slave and connect it with the Jenkins Master node:

![](cWzSbH4EqU50SVIzU1Hc3Q.webp)

From the first option, we will download the **agent.jar** file and then copy that file into the Slave node to the **/home/ec2-user** directory. As I am using MobaXterm I dragged the file into the home directory:

![](t7Y6XESkJk01dFGYob3VA.webp)

Once the file is copied we need to move this to **/opt directory :**

![](0GueRaJXC3WVVVuY_NIg.webp)

and from there we will execute the command as shown below:

```sh
curl -sO http://54.146.227.220:8080/jnlpJars/agent.jar  
java -jar agent.jar -jnlpUrl http://54.146.227.220:8080/manage/computer/Build%2DNode/jenkins-a
```

Output:

![](sjfj1JxQu3GInVBlb4kp8A.webp)

Now if we check our slave node it should be connected to our Master Jenkins Server:

![](MBxneIkttai6qprtObnSfg.webp)

## Step 4: Testing our Slave Node

Let’s create a simple jenkins job to test our slave node.

![](9x1yDmy9Gp2a02LdvfN-tg.webp)

Under **Build Steps** we are going to execute shell commands as shown below:

![](1CMHwUgc7yyfGlENKxuPQg.webp)

By default, this job will run on the master node however under **General** we will select **Restrict where this project can be run** option and mention the **label** we provided while configuring our slave node:

![](JFb1dVeL4uBieV1YOTCpdA.webp)

Click on Apply and Save to proceed.

Before building our new jo let’s verify the uptime on our Slave node:

```
[root@jenkins-slave ~]# uptime  
 19:55:44 up  3:02,  2 users,  load average: 0.46, 0.11, 0.04
```

Let's Build:

![](voADt1mQlvoFjW6RNATkCA.webp)

The Build has been successful, we can verify this new build in the workspace directory of our slave node:

![](4t8bbf-E_LJFta8KS8Hjrw.webp)

# Conclusion

In this blog, we learned how to configure and connect Slave Node to the Jenkins Master node and run a simple job on the Slave Node.
