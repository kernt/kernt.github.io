---
tags:
  - postgresql
  - datenbanken
---
## Postgresql Installation

Installation auf verschiedene Linux Distributionen.

### Installation auf ein Centos 7 OS.

```s
wget https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-redhat96-9.6-1.noarch.rpm
sudo rpm -ihv pgdg-redhat96-9.6-1.noarch.rpm
sudo yum -y install postgresql96-server postgresql96-devel postgresql96-contrib
sudo systemctl start postgresql-9.6
sudo systemctl status postgresql-9.6
sudo /usr/pgsql-9.6/bin/postgresql96-setup initdb
sudo systemctl enable postgresql-9.6
sudo systemctl start postgresql-9.6
```

**Datenbank initialisation**

`sudo postgresql-setup initdb`

**Service activities und starten**

`sudo systemctl enable postgresql && sudo systemctl start postgresql`

**Installation Sicherer machen**

```ss
su - postgres
psql -d template1 -c "ALTER USER postgres WITH PASSWORD 'newpassword';"
```

Password des Datenbank benutzers Postgres

# Posgres Administraion

PostgreSQL wird durch diesen Befehl installiert, wobei die Software automatisch den **Linux-Nutzer** mit dem Namen „postgres” für den Datenbankzugriff anlegt. Aus Sicherheitsgründen sollte dieser **lediglich für die Arbeit mit der Datenbank** genutzt werden. Außerdem ist es empfehlenswert, dieses Profil im ersten Schritt mit einem **Passwort** zu versehen (standardmäßig ist keines eingetragen). Hierfür muss man lediglich den folgenden Befehl und anschließend zwei Mal das Kennwort der Wahl eingeben:
```sh
sudo passwd postgres
```
Parallel zu dem Linux-User „postgres“ existiert der gleichnamige Datenbank-User, der für die **Administration der Datenbank** benötigt wird und ebenfalls ein starkes Passwort erhalten sollte. Das gelingt durch die Eingabe folgender Terminal-Befehle („_neues-passwort_“ ist dabei Platzhalter für das gewünschte Kennwort und entsprechend zu ersetzen):
```sh
su - postgres
psql -d template1 -c "ALTER USER postgres WITH PASSWORD 'neues-passwort';"
```
Um zu überprüfen, ob die **Installation erfolgreich** war, sollte man mit dem „postgres“-Linux-Account testweise eine Datenbank (hier: testdb) anlegen und mit dem **Terminal-Client psql** ansteuern:
```sh
su - postgres
createdb testdb
psql testdb
```
Im Terminal sieht man dann diesen **Output der psql-Client-Shell**, die sich mit beliebigen SQL-Kommandos bedienen lässt
## Fork BIGSQL mit Cluster

|Distribution|Zweck|Ort| Psql Version|
| :---: | :---: | :---: | :---: |
|Debain/Ubuntu|Basisverzeichnis|/opt/postgresql/$version/|9 BigSQL|
|CentOS|Basisverzeichnis|||
### Quellen zu BigSQL

* [BigSQL Debian](https://www.bigsql.org/docs/deb.jsp)
* [app-pgdump](https://www.postgresql.org/docs/9.3/app-pgdump.html)
# 1. Postgres as the Cluster Datastore for Kubernetes

Let’s start with something that’ll raise some eyebrows: using Postgres as the state store for your Kubernetes cluster. And no, I don’t mean running a PostgreSQL Helm chart _on_ Kubernetes. I’m talking about using a Postgres database as the cluster’s core datastore _instead of_ the default embedded etcd.

Typically, Kubernetes clusters use etcd as their primary key-value store for all the cluster’s critical data. But for certain environments or use cases, Postgres can be configured to play this role, providing a familiar relational database for Kubernetes’ operational needs. It might not be the first choice for most, but it’s a surprisingly robust alternative in specific scenarios!

![](https://miro.medium.com/v2/resize:fit:700/0*sHf4gYIIbwZNPF1j)

# 2. Postgres as the Backend for Helm Releases

Another fascinating use case comes from the world of Helm, the package manager for Kubernetes. Did you know that Postgres can be the backend database for your Helm releases?

Helm stores the details of your deployed releases, including configuration and chart information, in a persistent store. By using Postgres as this backend, you get a centralized, relational view of all your Helm deployments. This can be especially useful for larger teams who need more visibility into what’s deployed and for managing release history in a structured way.

![](https://miro.medium.com/v2/resize:fit:700/0*x3YcQ4TOaKJDyCxd)

Here is a link to a tutorial that I found on the internet: [tutorial](https://medium.com/linux-shots/use-postgresql-as-backend-storage-for-helm-de407cd9c43)

# 3. Promscale — PostgreSQL for Metrics, Tracing, and Logs

One of the coolest (though sadly discontinued) projects that used Postgres is **Promscale**. Even though Promscale is now read-only (cue crowd booing sound effect), its vision is still worth noting. Promscale allowed you to store prometheus metrics within Postgres for long-term storage.

![](https://miro.medium.com/v2/resize:fit:700/0*lVoeN94WtUaKEvWW)

This gave users a unified way to store all their observability data in a relational database, something that was previously unheard of. Promscale was particularly useful for those already using Postgres, as it eliminated the complexity of achieving long-term storage with Postgres. Even though Promscale is no longer actively maintained, it might still be worth exploring for smaller environments or those looking to streamline their data storage. And who knows — maybe some adventurous developers will fork it and continue the work one day.

![](https://miro.medium.com/v2/resize:fit:700/0*_in8NF_klP3rMn7o)

Promscale was part of the broader monitoring solution called **tobs**, which built on top of the popular `kube-prometheus-stack`. Tobs essentially provided a preconfigured stack for monitoring Kubernetes, with Promscale offering the unique Postgres-based storage.

# 4. Dapr: Using Postgres for Application Configuration

or those tired of the Kubernetes talk, here’s a more developer-friendly use case: **Dapr** (Distributed Application Runtime) allows you to use Postgres to store application configuration.

Yes, you read that right — Postgres isn’t just for relational data storage. With Dapr’s configuration APIs, you can use Postgres as a reliable, scalable store for your application’s configuration settings, making it easy to manage app configurations across environments and deployments.

![](https://miro.medium.com/v2/resize:fit:700/0*8eKtnDC-lkVd2mi2)

# 5. MassTransit and Postgres as a Message Queue

If you’re a .NET developer and a fan of **MassTransit** (the tool-agnostic message library), Postgres can also act as your message queue. Now, while Kafka or RabbitMQ might be the better choice on paper for large-scale systems, in the early stages of development, or for smaller user bases, Postgres does just fine for message handling.

![](https://miro.medium.com/v2/resize:fit:700/0*Hf14pgL4dQQiLmvM)

On top of that, you can leverage the same Postgres database to implement the **Outbox pattern**, ensuring reliable delivery of messages in distributed systems. The ability to use Postgres for multiple roles in your app infrastructure means fewer moving parts and a simplified setup.

![](https://miro.medium.com/v2/resize:fit:700/0*13nQEq4cPjDfk0or)

# 6. Projects That Use Postgres (Some You Might Not Expect)

- **AWX/Ansible Tower**: AWX, the open-source version of Ansible Tower, uses Postgres as its main database. Having deployed AWX in my homelab, I can confirm how smoothly it runs with Postgres as its data store.

![](https://miro.medium.com/v2/resize:fit:700/0*viv34jBF9gMEaT8w)

- **Unleash**: Yes, even **Unleash**, the popular feature flag project, uses Postgres as its backend. It’s a testament to Postgres’s reliability and scalability that projects requiring robust, consistent storage trust it for their core infrastructure.

![](https://miro.medium.com/v2/resize:fit:700/0*xfoH_3YUlIRa-iiF)

https://technotim.live/posts/postgresql-high-availability/