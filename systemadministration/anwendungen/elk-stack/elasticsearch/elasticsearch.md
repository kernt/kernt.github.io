---
tags:
  - elasticsearch
  - elk
  - elk-stack
---
# Elasticsearch

*/usr/share/elasticsearch/bin/elasticsearch*
## Elasticsearch absichern

Zur sicheren Kommunikation kann, certbot verwendet werden
Dazu muss die Konfigurationsdatei folgernd ermassen erweitert werden:

```
...
xpack.ssl.key: /etc/elasticsearch/ssl/vmd36612.contaboserver.net/privkey.pem
xpack.ssl.certificate: /etc/elasticsearch/ssl/vmd36612.contaboserver.net/fullchain.pem
xpack.ssl.certificate_authorities: [
  "/etc/elasticsearch/ssl/vmd36612.contaboserver.net/chain.pem",
  "/etc/elasticsearch/ssl/vmd36612.contaboserver.net/cacert.pem" ]

xpack.security.transport.ssl.enabled: true
xpack.security.http.ssl.enabled: true
xpack.security.audit.enabled: true

xpack.security.audit.enabled: true
xpack.security.audit.outputs: [ index, logfile ]
xpack.security.audit.logfile.events.include: [ access_denied, authentication_failed, connection_denied ]
```

> Info: Es kann sein das die _xpack_ Erweiterungen installiert werden müssen.
## Elasticsearch Konfiguration

_/etc/elasticsearch/elasticsearch.yml_

```
##################################
cluster.name: my-cluster
node.name: elastic-node1
path.data: /data/elastic/elastic-node1
path.logs: /var/log/elasticsearch

bootstrap.memory_lock: true
network.host: 192.168.1.1
http.port: 9200

# Make sure to add all your nodes host names
discovery.zen.ping.unicast.hosts: ["elastic-node1", "elastic-node2"]

# If we have more than 2 nodes in our cluster, this setting should be set
# to '2' or bigger number
discovery.zen.minimum_master_nodes: 1

##################################
```

> Die Heap-Größe muss 50 % des Server-Arbeitsspeichers betragen. Beispiel: In einem 24-GB-Elasticsearch-Knoten bedeutet dies, dass eine Heap-Größe von 12 GB optimale Leistung verspricht.
## Enable Elasticsearch on Debian After Installation

Post-installation, the Elasticsearch service is disabled on boot and inactive by default. To enable the Elasticsearch service and set it to start automatically at boot, use the following `systemctl` command:

```bash
sudo systemctl enable elasticsearch.service --now
```

## Configure Elasticsearch 8

The next stage of managing Elasticsearch 8 on your Debian Linux system is configuring it.

### Getting Acquainted with Default Elasticsearch Settings

On a fresh Elasticsearch installation, the software places its processed and stored data within the /var/lib/elasticsearch directory. If you need configuration modifications, /etc/elasticsearch is your go-to directory. If Java start-up options need tweaking, these settings can be adjusted in the /etc/default/elasticsearch configuration file.

These standard settings fit standalone servers where Elasticsearch operates exclusively on localhost. However, if your sights are set on establishing an Elasticsearch cluster or permitting remote connections, modifications to the default configuration are necessary.

### Refining the Elasticsearch Configuration File

The Elasticsearch configuration file contains many parameters that can be adjusted to meet your unique needs. To open this file, execute the following command:

```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```

An important parameter is cluster.name, which designates the cluster name to which your node belongs. By default, this is “elasticsearch”, but a more distinct name is advisable in a production environment.

Here’s an example:

```
cluster.name: my_application
```

The node.name option is another significant one that sets the name of the Elasticsearch node. This name is essential for smooth administration and management, especially when working with multiple nodes.

Here’s an example:

```
node.name: node-1
```

### Configure Elasticsearch 8 with HTTPS on Debian

The following covers configuring HTTPS for your Elasticsearch installation. Encryption is essential to securing data, and enabling Elasticsearch 8 to use HTTPS ensures data is securely transmitted between nodes and clients.

#### Generate SSL Certificates

The first step in configuring Elasticsearch for HTTPS is to generate an SSL certificate. This certificate is what’s going to encrypt the data between your server and clients. Elasticsearch has a built-in tool called elasticsearch-certutil that you can use to generate a self-signed certificate. Run the following command:

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-certutil cert --silent --pem -out config/elastic-certificates.p12
```

This will create a .p12 file in the config directory. This file contains both the private key and the public certificate. It is essential to protect this file and ensure it’s not accessible to unauthorized users.

#### Update Elasticsearch 8 Configuration

Next, you need to update the Elasticsearch configuration file located at /etc/elasticsearch/elasticsearch.yml to include the paths to your certificate and private key. Open the configuration file and add the following lines:

```
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
```

These settings enable SSL for the transport layer, specify that we use certificates for SSL verification, and define the paths to the certificate and private key.

### Allowing Remote Access (Optional)

Elasticsearch is configured to listen only to localhost by default. However, this can be adjusted in cases where remote access is required. Look for the Network section and uncomment the line network.host by removing the # in front. Replace the existing value with your internal private IP address or external IP address.

Here’s an example:

```
# network.host: 192.168.0.1
```

In this example, network.host has been uncommented and adjusted to an internal private IP address.

For security, specifying individual IP addresses is advisable. However, you could change the network interface to listen to all by setting it to 0.0.0.0 for multiple internal or external IP addresses connecting to the server.

After making the necessary changes, save the configuration file (CTRL+O to save, CTRL+X to exit).

To enforce the configuration file changes, restart the Elasticsearch service with the command:

```bash
sudo systemctl restart elasticsearch
```

### Modifying the UFW Firewall Rules for Remote Connections

If you’ve configured Elasticsearch to allow remote connections, adjusting your firewall rules to accommodate these connections is vital. The Uncomplicated Firewall (UFW) can assist with this.

You can permit a specific IP address to connect to Elasticsearch by executing this command:

```bash
sudo ufw allow from <IP Address> to any port 9200
```

Don’t forget to replace IP Address with the actual IP address you wish to allow connections from. This command will open port 9200 for the specified IP address, permitting it to interact with Elasticsearch.

For example, if you wish to allow the IP address 192.168.0.2 to connect, the command would be:

```bash
sudo ufw allow from 192.168.0.2 to any port 9200
```

Following this, your Debian server will permit traffic from 192.168.0.2 to access Elasticsearch through port 9200. Remember that the security of your servers is paramount, so ensure that you only grant access to trusted IP addresses.
## Verifying Elasticsearch 8 Configuration

After modifying your Elasticsearch configuration, it is vital to verify that the changes have been implemented correctly and that your Elasticsearch instance is functioning as expected.

### Checking the Elasticsearch 8 Service Status

First, check the status of your Elasticsearch service to confirm that it’s active and running. You can do this with the following command:

```bash
sudo systemctl status elasticsearch
```

This command will output information about the Elasticsearch service, including whether it’s active and running. If not, you may need to troubleshoot the issue or review your configuration changes.

### Testing Remote Access (Optional)

If you’ve enabled remote access, you can test it by trying to connect to your Elasticsearch instance from a remote machine. As discussed in the previous section, the IP address you’re connecting from must be allowed in your firewall rules.

You can use the curl command for this purpose:

```bash
curl https://<Your_Elasticsearch_IP>:9200
```

Replace <Your_Elasticsearch_IP> with the IP address of your Elasticsearch server. If successful, this command should return information about your Elasticsearch instance.

### Validating Data Integrity

Finally, it’s essential to verify the integrity of your Elasticsearch data, especially if you’ve made changes to the data directory in your configuration. Elasticsearch provides APIs that you can use to check the status of your data.

For example, to get the status of all indices, use the following command:

```bash
curl -X GET "https://localhost:9200/_cat/indices?v=true&pretty"
```

This command lists all indices in your Elasticsearch instance and their health status. Check the health status of your indices to ensure that your data is safe and accessible.

Note: Remember, maintaining the integrity and security of your data should always be a top priority. Always double-check your changes, validate your configuration, and monitor your Elasticsearch instance regularly. This will help you maintain a reliable, efficient, and secure data management system with Elasticsearch.

## Interacting with Elasticsearch 8 via cURL

This section will explore common commands that interact with your Elasticsearch 8 instance. We’ll use the `curl` command line tool, a flexible library for transferring data using different protocols.

### Deleting an Index

An Elasticsearch index is a set of documents that have similar characteristics. If you have an index named samples that you wish to delete, you can accomplish that with the following command:

```bash
curl -X DELETE 'https://localhost:9200/samples'
```

This command sends an HTTP DELETE request to the specified URL, which tells Elasticsearch to delete the samples index.

### Listing all Indices

To retrieve a list of all indices in your Elasticsearch instance, use the _cat/indices endpoint with a GET request like this:

```bash
curl -X GET 'https://localhost:9200/_cat/indices?v'
```

The response will list all indices, including the index’s health, status, size, and number of documents.

### Fetching Documents from an Index

To fetch all documents from a specific index, say sample, use the _search endpoint:

```bash
curl -X GET 'https://localhost:9200/sample/_search'
```

This command retrieves all documents stored in the sample index.

### Searching with URL Parameters

You can use Lucene’s query syntax for basic text searches. For instance, if you wanted to find documents where the school field is Harvard, you would use:

```bash
curl -X GET https://localhost:9200/samples/_search?q=school:Harvard
```

This command sends a GET request to the _search endpoint using the q parameter for the query.

### Searching with Elasticsearch Query DSL

For more complex queries, you might prefer to use Elasticsearch’s Query DSL, which allows you to use JSON to define queries. This format is more readable and more accessible for debugging complex queries.

Here’s an example:

```bash
curl -XGET --header 'Content-Type: application/json' https://localhost:9200/samples/_search -d '{
      "query" : {
        "match" : { "school": "Harvard" }
    }
}'
```

This command does the same as the previous one but uses JSON for the query definition. This way, the command is more readable, especially when dealing with more complex queries.

### Adding and Updating Documents

To add a new document to an index, you can use the PUT method along with the _doc endpoint. For example, to add a document with an ID of 1 and a school field of Harvard to the samples index, you would use:

```bash
curl -XPUT --header 'Content-Type: application/json' https://localhost:9200/samples/_doc/1 -d '{
   "school" : "Harvard"
}'
```

To update a document, you can use the POST method with the _update endpoint. For instance, to add a students field to the document we just created, you would use:

```bash
curl -XPOST --header 'Content-Type: application/json' https://localhost:9200/samples/_doc/1/_update -d '{
"doc" : {
               "students": 50000}
}'
```

This command tells Elasticsearch to update the document with an ID of 1 in the samples index, adding a students field with a value of 50000.

These commands are only a tiny sample of what you can accomplish with Elasticsearch. The Elasticsearch Query DSL

## Manage Elasticsearch 8 on Debian 12, 11 or 10

This segment delves into the Elasticsearch 8 removal process from your Debian Linux server. You may want to do this for several reasons, such as the software no longer suits your requirements or you’re planning on switching to an alternative solution. Regardless of your reasons, the steps outlined below will walk you through the process.

### Elasticsearch 8 Removal From Debian

Eradicating Elasticsearch from your server involves a simple command. However, remember that this action is irreversible and will delete all associated data. Here’s the command to execute:

```bash
sudo apt remove elasticsearch
```

Running the above command will effectively uninstall Elasticsearch, freeing up any previously allocated system resources.

#### Remove Elasticsearch APT Repository

The Elasticsearch repository will remain on your system after uninstallation. If you’re sure you won’t reinstall Elasticsearch in the future, you might as well get rid of it to avoid unnecessary clutter.

Here’s how to remove the Elasticsearch repository:

```bash
sudo rm /etc/apt/sources.list.d/elasticsearch-8.list
```

The above command will delete the Elasticsearch 8 repository from your Debian Linux server.

#### Refreshing Repository List

The final step is to refresh the apt package list. This step is essential to ensure your system doesn’t consider Elasticsearch 8 for future updates or installations.

The command to update your repository list is as follows:

```bash
sudo apt update
```

Quellen

* [elasticsearch documentation](https://www.netiq.com/de-de/documentation/sentinel-81/install/data/b1m3gtdt.html)
