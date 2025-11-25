
Wenn du deinen Kubernetes-Cluster ohne einen Cloud-Provider (also in einer On-Premise-Umgebung oder auf selbst verwalteten Servern) betreibst und eine Webseite im öffentlichen Internet verfügbar machen möchtest, kannst du dies ebenfalls tun. Der Prozess ist ähnlich wie bei einem Cloud-Provider, jedoch musst du einige Dinge selbst einrichten, wie zum Beispiel die IP-Adressen und das Routing. Hier sind die Schritte, um dies zu erreichen:
### 1. **Kubernetes-Cluster vorbereiten**

Stelle sicher, dass dein Kubernetes-Cluster ordnungsgemäß eingerichtet und funktionsfähig ist. Wenn du das Cluster auf physischen Servern oder virtuellen Maschinen betreibst, achte darauf, dass du Zugriff auf alle notwendigen Maschinen hast.
### 2. **Webanwendung in Kubernetes bereitstellen**

Zuerst musst du deine Webanwendung (z.B. eine einfache HTML-Seite, ein Nginx-Server oder eine komplexe Webanwendung) als Container erstellen und in Kubernetes bereitstellen.

Beispiel: Bereitstellung einer einfachen Nginx-Webseite

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

```
kubectl apply -f deployment.yaml
```
### 3. **Service für öffentliche Erreichbarkeit erstellen**

Um deine Anwendung für den öffentlichen Internetverkehr zugänglich zu machen, musst du einen Service einrichten. Da du keinen Cloud-Provider hast, wirst du vermutlich mit einem **NodePort**-Service arbeiten, der den Service auf einem Port des Nodes (deines Servers) verfügbar macht.
#### Beispiel eines NodePort-Service:

```
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80       # Der Port innerhalb des Clusters
      targetPort: 80 # Der Port des Containers
      nodePort: 30080 # Der Port, über den die Webseite von außen erreichbar ist
  type: NodePort
```

`kubectl apply -f service.yaml`

Dieser Service macht deine Webseite über den Port `30080` auf jedem Knoten (Node) in deinem Cluster verfügbar. Du kannst dann die IP-Adresse des Nodes verwenden, um darauf zuzugreifen (z.B. `http://<Node-IP>:30080`).
### 4. **Öffentlich zugängliche IP-Adresse**

Um deine Webseite tatsächlich über das öffentliche Internet zugänglich zu machen, musst du sicherstellen, dass der Port 30080 (oder der von dir gewählte Port) von außen erreichbar ist. Dafür gibt es mehrere Optionen:

- **Portweiterleitung auf deinem Router**: Wenn du deinen Cluster hinter einem Router betreibst (z.B. in einem Heimnetzwerk oder einem Rechenzentrum), musst du die Portweiterleitung (Port Forwarding) auf deinem Router einrichten, um Anfragen, die an eine öffentliche IP-Adresse auf einem bestimmten Port gesendet werden, an den Node in deinem Cluster weiterzuleiten.

- **Firewall konfigurieren**: Stelle sicher, dass die Firewall-Regeln auf den Knoten des Clusters so konfiguriert sind, dass sie eingehenden Traffic auf dem gewählten Port zulassen.

- **Öffentliche IP-Adresse**: Wenn du über eine feste öffentliche IP-Adresse verfügst, kannst du diese IP-Adresse verwenden, um auf deine Webseite zuzugreifen. Wenn du keine feste öffentliche IP hast, kannst du möglicherweise eine dynamische DNS-Lösung verwenden, die es dir ermöglicht, eine Domain auf eine dynamische IP zu zeigen.
### 5. **Optional: Ingress Controller (für bessere Verwaltung)**

Falls du mehrere Webseiten oder zusätzliche Funktionen wie SSL-Terminierung benötigst, kannst du ein **Ingress**-Controller verwenden. Ingress ermöglicht es, HTTP(S)-Verkehr auf mehrere Services innerhalb des Clusters zu leiten.
#### Beispiel für die Ingress-Ressource:

Zuerst musst du einen Ingress-Controller installieren, wie den **NGINX Ingress Controller**. Danach kannst du eine Ingress-Ressource definieren, um den eingehenden Verkehr zu deinen Anwendungen zu leiten.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  rules:
  - host: deine-domain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

Du musst dann sicherstellen, dass der **Ingress Controller** den Verkehr auf dem entsprechenden Port akzeptiert und weiterleitet.
### 6. **DNS einrichten**

Wenn du eine Domain verwenden möchtest (z.B. `deine-domain.com`), musst du einen DNS-Eintrag einrichten, der auf die öffentliche IP-Adresse deines Servers verweist. Du kannst entweder die IP des Nodes verwenden, der den NodePort Service verwaltet, oder bei Verwendung eines LoadBalancers die IP dieses LoadBalancers angeben.

- Gehe zu deinem DNS-Anbieter (z.B. GoDaddy, Namecheap) und erstelle einen **A-Record**, der auf die IP-Adresse deines Servers zeigt.
### Zusammenfassung:

1. **Webanwendung im Kubernetes Cluster bereitstellen**.
2. **NodePort-Service einrichten**, um den öffentlichen Zugriff auf den Cluster zu ermöglichen.
3. **Portweiterleitung und Firewall konfigurieren**, damit der Dienst öffentlich zugänglich ist.
4. (Optional) **Ingress Controller und DNS** konfigurieren, um den Traffic besser zu steuern und die Anwendung benutzerfreundlicher zu machen.

Mit diesen Schritten kannst du eine Webseite über deinen eigenen Kubernetes-Cluster im Internet verfügbar machen, ohne einen Cloud-Provider zu benötigen.

https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal-clusters

Given the following 3-node Kubernetes cluster the external IP is added as an example, in most bare-metal environments this value is None

```
$ kubectl get node
NAME     STATUS   ROLES    EXTERNAL-IP
host-1   Ready    master   203.0.113.1
host-2   Ready    node     203.0.113.2
host-3   Ready    node     203.0.113.3
```

and the following `ingress-nginx` NodePort Service

```
$ kubectl -n ingress-nginx get svc
NAME                   TYPE        CLUSTER-IP     PORT(S)
ingress-nginx          NodePort    10.0.220.217   80:30100/TCP,443:30101/TCP
```

One could set the following external IPs in the Service spec, and NGINX would become available on both the NodePort and the Service port:

```
spec:
  externalIPs:
    - 62.171.183.4
    - 144.91.77.182
    - 86.48.5.122
```

```sh
curl -D- http://myapp.example.com:30100
HTTP/1.1 200 OK
Server: nginx/1.15.2
```

```sh
curl -D- http://vmd132643.contaboserver.net && http://vmd134720.contaboserver.net && http://vmd154798.contaboserver.net
HTTP/1.1 200 OK
Server: nginx/1.15.2
```

We assume the myapp.example.com subdomain above resolves to both 203.0.113.2 and 203.0.113.3 IP addresses.
