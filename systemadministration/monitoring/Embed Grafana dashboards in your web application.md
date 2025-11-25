# What is Grafana?

Grafana is a multi-platform open source analytics and visualization tool. It provides charts, graphs, and alerts for the web Applications.Grafana connects with every possible data source, commonly referred to as databases such as _Graphite, Prometheus, Influx DB, ElasticSearch, MySQL, PostgreSQL_ etc and various With various cloud Services as well such as AWS cloudwatch.This framework has gained a lot of popularity in very short time and being used by big guns such as PayPal, eBay, Intel & many more.

# Why it is so famous?

1. We can monitor our applications easily help of cool customizable dashboards.
2. Grafana connects with lot many data sources.It is very easy to import data from this many data sources to Grafana.
3. Grafana being an open source solution also enables us to write plugins from scratch to integrate with other data sources.

In this Article I have explained How we can integrate Dashboard which we have created in Grafana into our Web Application. I have create a Simple Django Application and inside that I have integrated Grafana Dashboard.

First we need Grafana to be installed ,you may used VM or docker as well.I have installed it on my Local machine and I am using RHEL 8.

For RHEL Lets get the latest RPM and install it.You can find installation steps for each OS in official Grafana documentation.

```
wget [https://dl.grafana.com/oss/release/grafana-8.0.1-1.x86_64.rpm](https://dl.grafana.com/oss/release/grafana-8.0.1-1.x86_64.rpm);  
sudo yum install grafana-8.0.1-1.x86_64.rpm
```

Once done you can check if Grafana service is Running if not then use below command to start Grafana Service.

```sh
[root@localhost ~]# systemctl restart grafana-server.service  
[root@localhost ~]# systemctl status grafana-server.service  
● grafana-server.service - Grafana instance  
   Loaded: loaded (/usr/lib/systemd/system/grafana-server.service; disabled; vendor preset: disabled)  
   Active: active (running) since Fri 2021-06-11 16:43:57 IST; 8s ago  
     Docs: [http://docs.grafana.org](http://docs.grafana.org)  
 Main PID: 9600 (grafana-server)  
    Tasks: 6 (limit: 11328)  
   Memory: 34.2M  
   CGroup: /system.slice/grafana-server.service  
           └─9600 /usr/sbin/grafana-server --config=/etc/grafana/grafana.ini --pidfile=/var/run/grafana/grafana-server.pid --packaging=rpm cfg:default.paths.logs=/var/
```

Now Grafana is Setup and installed successfully. Grafana uses default 3000 port but you can change it by changing port value into **/etc/grafana/grafana.ini** which is main configuration File for Grafana. Now you can check if grafana is opening or not using http://hostname:port url. example. [http://127.0.0.1:3000/](http://127.0.0.1:3000/)(default username and password is admin/admin).

Now Lets create a Simple dashboard in Grafana.Click on the DASHBOARDS(Create your first dashboard) Section From Grafana.

Now lets add a empty panel by clicking on the add an empty panel section.

![](HT23gWxXpmixLRXHsBhNpw.webp)

You can now see Simple Dashboard in the Grafana which we will import into our Web App.

![](JDTY_fXn0SIJZeNVXWW92Q.webp)

You can embed a panel using an iframe on Web Application. Unless anonymous access permission is enabled, the viewer must be signed into Grafana to view the graph.

We need to change some parameters from default Grafana Configuration file(**/etc/grafana/grafana.ini**) for sharing it with our django Application. Change below two paramaters in default configuration file. Please note: **you need to restart grafana service each time you change in the configuration file.**Then and then only grafana will load your latest configurations.You can cross verify the changes by visiting (Server admin -> setting)section from Grafana. Keep a note that Semicolons (the `;` char) are the standard way to comment out lines in a `.ini` file. If you want to change a setting, you must delete the semicolon (`;`) in front of the setting before it will work.

```sh
allow_embedding = true  
[auth.anonymous]  
# enable anonymous access  
enabled = true
```

Login into Grafana dashboard and Select the dashboard which You want to embed. From the dashboard click on (Panel Title -> Share -> Embed). Copy the HTML code shown below.

![](nOBMe9IqGJeuCzE-m_YfRw.webp)

![](98ZhKFJ5JN9x2_viixZCsw.webp)

You can now go into your source code and paste the copied HTML code into your HTML page.This is a Simple iframe HTML tag as shown below.

```xml
<iframe src="[http://localhost:3001/d-solo/KBTvj66Mk/new-dashboard?orgId=1&from=1623380741011&to=1623402341011&panelId=2](http://localhost:3001/d-solo/KBTvj66Mk/new-dashboard?orgId=1&from=1623380741011&to=1623402341011&panelId=2)" width="450" height="200" frameborder="0">
</iframe>
```

Now You are done with all the steps. You can go ahead and check if its displaying Grafana dashboard correctly in Your Application.

![](wpOhduA9kJzTiJrgrg8XVA.webp)

