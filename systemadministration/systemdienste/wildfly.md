---
tags:
  - java
  - wildfly
---
# WildFly Konfigurartion

## Ports einrichten

Unter Windows:

`standalone.bat -Djboss.http.port=<Desired_Port_Number>`

Unter Unix/Linux:

`standalone.sh -Djboss.http.port=<Desired_Port_Number>`

Quellen:

* [wildfly-change-default-port](https://www.baeldung.com/wildfly-change-default-port)
## mod_cluser

### mod_cluster Konfiguration

Konfiguration der Datei _mod_cluster.conf_

```
# Note: There is only "Listen 192.168.122.204:2181" in conf/http.conf,
#       all other configuration including SSL is done here for demonstration purposes.

LoadModule slotmem_module modules/mod_slotmem.so
LoadModule manager_module modules/mod_manager.so
LoadModule proxy_cluster_module modules/mod_proxy_cluster.so
LoadModule advertise_module modules/mod_advertise.so

MemManagerFile "/dev/shm/httpd/cache/mod_cluster"

# mod_cluster directive for emulating cpin/cpong
EnableOptions

# SSL properties for vhost on 2181
SSLEngine on
SSLProtocol all -SSLv2 -SSLv3
SSLCipherSuite "EECDH+ECDSA+AESGCM EECDH+aRSA+AESGCM EECDH+ECDSA+SHA384 EECDH+ECDSA+SHA256 EECDH+aRSA+SHA384 EECDH+aRSA+SHA256 EECDH+aRSA+RC4 EECDH EDH+aRSA !RC4 !aNULL !eNULL !LOW !3DES !MD5 !EXP !PSK !SRP !DSS"
SSLHonorCipherOrder on
SSLCertificateFile /vault/certs/server.crt
SSLCertificateKeyFile /vault/certs/server.key
SSLCACertificateFile /vault/certs/myca.crt
SSLVerifyClient require
SSLProxyVerify require
SSLProxyEngine On
SSLVerifyDepth 10
SSLProxyMachineCertificateFile /vault/certs/client.pem
SSLProxyCACertificateFile /vault/certs/myca.crt
SSLProxyProtocol all -SSLv2 -SSLv3

ServerName 192.168.122.204:2181

# MOD_CLUSTER
<IfModule manager_module>
  Listen 192.168.122.204:8847
  # Test and demonstration purposes...
  LogLevel debug
  <VirtualHost 192.168.122.204:8847>
    ServerName 192.168.122.204:8847
    <Directory />
      Order deny,allow
      Deny from all
      [[CHANGEIT]] This is only for testing!
      Allow from all
    </Directory>
    KeepAliveTimeout 60
    MaxKeepAliveRequests 0
    ServerAdvertise on
    AdvertiseFrequency 5
    ManagerBalancerName qacluster
    AdvertiseGroup 224.0.5.79:65009
    EnableMCPMReceive

    # SSL properties for mod_cluster vhost

    SSLEngine on
    SSLProtocol all -SSLv2 -SSLv3
    SSLCipherSuite "EECDH+ECDSA+AESGCM EECDH+aRSA+AESGCM EECDH+ECDSA+SHA384 EECDH+ECDSA+SHA256 EECDH+aRSA+SHA384 EECDH+aRSA+SHA256 EECDH+aRSA+RC4 EECDH EDH+aRSA !RC4 !aNULL !eNULL !LOW !3DES !MD5 !EXP !PSK !SRP !DSS"
    SSLHonorCipherOrder on
    SSLCertificateFile /vault/certs/server.crt
    SSLCertificateKeyFile /vault/certs/server.key
    SSLCACertificateFile /vault/certs/myca.crt
    SSLVerifyClient require
    SSLProxyVerify require
    SSLProxyEngine On
    SSLVerifyDepth 10
    SSLProxyMachineCertificateFile /vault/certs/client.pem
    SSLProxyCACertificateFile /vault/certs/myca.crt
    SSLProxyProtocol all -SSLv2 -SSLv3

    <Location /mcm>
      SetHandler mod_cluster-manager
      Order deny,allow
      Deny from all
      [[CHANGEIT]] This is only for testing!
      Allow from all
    </Location>
  </VirtualHost>
</IfModule>
```

Konfiguration der Datei _standalone-ha.xml_

```xml
<subsystem xmlns="urn:jboss:domain:web:2.1" native="false">
<connector name="https" protocol="HTTP/1.1" socket-binding="https" scheme="https" enabled="true" secure="true">
<ssl name="https" 
    ca-certificate-file="/vault/certs/ca-cert.jks" 
    certificate-key-file="/vault/certs/server-cert-key.jks"
    certificate-file="/vault/certs/server-cert-key.jks"
    password="tomcat"
    verify-client="true"
    key-alias="javaserver"
    cipher-suite="SSL_DHE_RSA_WITH_3DES_EDE_CBC_SHA,SSL_RSA_WITH_3DES_EDE_CBC_SHA,SSL_DHE_DSS_WITH_3DES_EDE_CBC_SHA" protocol="TLSv1"/>
</connector>
<virtual-server name="default-host" enable-welcome-root="true">
    <alias name="localhost"/>
    <alias name="example.com"/>
</virtual-server>
</subsystem>

```

Quellen:

* [mod_cluster](https://docs.modcluster.io/)
* [modcluster.io examples/](https://modcluster.io/examples/)
* [red_hat_jboss mod_cluster](https://access.redhat.com/documentation/en-us/red_hat_jboss_operations_network/3.0/html/manage_jboss_servers/mod_cluster)
* [how-to-install-wildfly-on-centos-7](* [](https://linuxize.com/post/how-to-install-wildfly-on-centos-7/))