
jitsi Depoyment mit Docker

```sh
wget $(curl -s https://api.github.com/repos/jitsi/docker-jitsi-meet/releases/latest | grep 'zip' | cut -d\" -f4)

unzip *.zip
cp env.example .env
./gen-passwords.sh
mkdir -p ~/.jitsi-meet-cfg/{web,transcripts,prosody/config,prosody/prosody-plugins-custom,jicofo,jvb,jigasi,jibri}
```

edit .env

```
# shellcheck disable=SC2034

################################################################################
################################################################################
# Welcome to the Jitsi Meet Docker setup!
#
# This sample .env file contains some basic options to get you started.
# The full options reference can be found here:
# https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-docker
################################################################################
################################################################################

#
# Basic configuration options
#

# Directory where all configuration will be stored
CONFIG=~/.jitsi-meet-cfg

# Exposed HTTP port (will redirect to HTTPS port)
HTTP_PORT=8000

# Exposed HTTPS port
HTTPS_PORT=8443

# System time zone
TZ=UTC

# Public URL for the web service (required)
# Keep in mind that if you use a non-standard HTTPS port, it has to appear in the public URL
#PUBLIC_URL=https://meet.example.com:${HTTPS_PORT}

# Media IP addresses to advertise by the JVB
# This setting deprecates DOCKER_HOST_ADDRESS, and supports a comma separated list of IPs
# See the "Running behind NAT or on a LAN environment" section in the Handbook:
# https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-docker#running-behind-nat-or-on-a-lan-environment
#JVB_ADVERTISE_IPS=192.168.1.1,1.2.3.4

#
# Memory limits for Java components
#

#JICOFO_MAX_MEMORY=3072m
#VIDEOBRIDGE_MAX_MEMORY=3072m

#
# JaaS Components (beta)
# https://jaas.8x8.vc
#

# Enable JaaS Components (hosted Jigasi)
# NOTE: if Let's Encrypt is enabled a JaaS account will be automatically created, using the provided email in LETSENCRYPT_EMAIL
#ENABLE_JAAS_COMPONENTS=0

#
# Let's Encrypt configuration
#

# Enable Let's Encrypt certificate generation
#ENABLE_LETSENCRYPT=1

# Domain for which to generate the certificate
LETSENCRYPT_DOMAIN=meet.tuxcoach.de

# E-Mail for receiving important account notifications (mandatory)
LETSENCRYPT_EMAIL=tobkern1980@gmail.com

# Use the staging server (for avoiding rate limits while testing)
#LETSENCRYPT_USE_STAGING=1

# Set ACME server. Default is zerossl, you can peek one at https://github.com/acmesh-official/acme.sh/wiki/Server
#LETSENCRYPT_ACME_SERVER="letsencrypt"

#
# Etherpad integration (for document sharing)
#

# Set the etherpad-lite URL in the docker local network (uncomment to enable)
#ETHERPAD_URL_BASE=http://etherpad.meet.jitsi:9001

# Set etherpad-lite public URL, including /p/ pad path fragment (uncomment to enable)
#ETHERPAD_PUBLIC_URL=https://etherpad.tuxcoach.de/p/

#
# Whiteboard integration
#

# Set the excalidraw-backend URL in the docker local network (uncomment to enable)
#WHITEBOARD_COLLAB_SERVER_URL_BASE=http://whiteboard.meet.jitsi.tuxcoach.de

# Set the excalidraw-backend public URL (uncomment to enable)
#WHITEBOARD_COLLAB_SERVER_PUBLIC_URL=https://whiteboard.tuxcoach.de


#
# Basic Jigasi configuration options (needed for SIP gateway support)
#

# SIP URI for incoming / outgoing calls
#JIGASI_SIP_URI=test@sip2sip.info

# Password for the specified SIP account as a clear text
#JIGASI_SIP_PASSWORD=passw0rd

# SIP server (use the SIP account domain if in doubt)
#JIGASI_SIP_SERVER=sip2sip.info

# SIP server port
#JIGASI_SIP_PORT=5060

# SIP server transport
#JIGASI_SIP_TRANSPORT=UDP

#
# Authentication configuration (see handbook for details)
#

# Enable authentication (will ask for login and password to join the meeting)
#ENABLE_AUTH=1

# Enable guest access (if authentication is enabled, this allows for users to be held in lobby until registered user lets them in)
#ENABLE_GUESTS=1

# Select authentication type: internal, jwt, ldap or matrix
#AUTH_TYPE=internal

# JWT authentication
#

# Application identifier
#JWT_APP_ID=my_jitsi_app_id

# Application secret known only to your token generator
#JWT_APP_SECRET=my_jitsi_app_secret

# (Optional) Set asap_accepted_issuers as a comma separated list
#JWT_ACCEPTED_ISSUERS=my_web_client,my_app_client

# (Optional) Set asap_accepted_audiences as a comma separated list
#JWT_ACCEPTED_AUDIENCES=my_server1,my_server2

# LDAP authentication (for more information see the Cyrus SASL saslauthd.conf man page)
#

# LDAP url for connection
#LDAP_URL=ldaps://ldap.tuxcoach.de/

# LDAP base DN. Can be empty
#LDAP_BASE=DC=example,DC=tuxcoach,DC=de

# LDAP user DN. Do not specify this parameter for the anonymous bind
#LDAP_BINDDN=CN=binduser,OU=users,DC=example,DC=tuxcoach,DC=de

# LDAP user password. Do not specify this parameter for the anonymous bind
#LDAP_BINDPW=LdapUserPassw0rd

# LDAP filter. Tokens example:
# %1-9 - if the input key is user@mail.domain.com, then %1 is com, %2 is domain and %3 is mail
# %s - %s is replaced by the complete service string
# %r - %r is replaced by the complete realm string
#LDAP_FILTER=(sAMAccountName=%u)

# LDAP authentication method
#LDAP_AUTH_METHOD=bind

# LDAP version
#LDAP_VERSION=3

# LDAP TLS using
#LDAP_USE_TLS=1

# List of SSL/TLS ciphers to allow
#LDAP_TLS_CIPHERS=SECURE256:SECURE128:!AES-128-CBC:!ARCFOUR-128:!CAMELLIA-128-CBC:!3DES-CBC:!CAMELLIA-128-CBC

# Require and verify server certificate
#LDAP_TLS_CHECK_PEER=1

# Path to CA cert file. Used when server certificate verify is enabled
#LDAP_TLS_CACERT_FILE=/etc/ssl/certs/ca-certificates.crt

# Path to CA certs directory. Used when server certificate verify is enabled
#LDAP_TLS_CACERT_DIR=/etc/ssl/certs

# Wether to use starttls, implies LDAPv3 and requires ldap:// instead of ldaps://
# LDAP_START_TLS=1


#
# Security
#
# Set these to strong passwords to avoid intruders from impersonating a service account
# The service(s) won't start unless these are specified
# Running ./gen-passwords.sh will update .env with strong passwords
# You may skip the Jigasi and Jibri passwords if you are not using those
# DO NOT reuse passwords
#

# XMPP password for Jicofo client connections
JICOFO_AUTH_PASSWORD=24ca56e1537b28ac384ee1a7e7042336

# XMPP password for JVB client connections
JVB_AUTH_PASSWORD=7629185d10d4977551cfad5249f1ca35

# XMPP password for Jigasi MUC client connections
JIGASI_XMPP_PASSWORD=4cd77acf3ef9c6cc1b011e3f8b4c9cfa

# XMPP password for Jigasi transcriber client connections
JIGASI_TRANSCRIBER_PASSWORD=9955eeaf041d7abdaf57cf4a6590798a

# XMPP recorder password for Jibri client connections
JIBRI_RECORDER_PASSWORD=2ea2199b7797f6b675cb5218eead9a62

# XMPP password for Jibri client connections
JIBRI_XMPP_PASSWORD=e5c5e081fb23b2ae9177703838e09b29

#
# Docker Compose options
#

# Container restart policy
#RESTART_POLICY=unless-stopped

# Jitsi image version (useful for local development)
#JITSI_IMAGE_VERSION=latest
```

> Access the web UI at [`https://localhost:8443`](https://localhost:8443) (or a different port, in case you edited the `.env` file)

If you want to use jigasi too, first configure your env file with SIP credentials and then run Docker Compose as follows:

```
docker compose -f docker-compose.yml -f jigasi.yml up
```

If you want to enable document sharing via [Etherpad](https://github.com/ether/etherpad-lite), configure it and run Docker Compose as follows:

```
docker compose -f docker-compose.yml -f etherpad.yml up
```

If you want to use jibri too, first configure a host as described in Jitsi Broadcasting Infrastructure configuration section and then run Docker Compose as follows:

```
docker compose -f docker-compose.yml -f jibri.yml up -d
```

or to use jigasi too:

```
docker compose -f docker-compose.yml -f jigasi.yml -f jibri.yml up -d
```

To include a transcriber component, run Docker Compose as follows:

```
docker compose -f docker-compose.yml -f transcriber.yml up -d
```

Or for them all together:

```
docker compose -f docker-compose.yml -f transcriber.yml -f jigasi.yml -f jibri.yml up -d
```

For the log analysis project, you will need both log-analyser.yml and grafana.yml files. This project allows you to analyze docker logs in grafana. If you want to run the log analyzer, run the Docker files as follows:

```
docker-compose -f docker-compose.yml -f log-analyser.yml -f grafana.yml up -d
```

Follow [this](https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-log-analyser) document for detailed information on log analysis.
