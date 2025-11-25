
```sh
curl -s https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh | sudo bash
```

```sh
sudo apt install crowdsec
```

```sh
sudo systemctl status crowdsec
```

```sh
crowdsec (1.6.9) wird eingerichtet ...
debconf: kann Oberfläche nicht initialisieren: Dialog
debconf: (Für die Dialog-Oberfläche muss das Bild mindestens 13 Zeilen hoch und 31 Spalten breit sein.)
debconf: greife zurück auf die Oberfläche: Readline
Creating /etc/crowdsec/acquis.yaml
INFO[2025-06-24 19:15:52] crowdsec_wizard: service 'apache2': /var/log/apache2/error.log
INFO[2025-06-24 19:15:52] crowdsec_wizard: using journald for 'nginx'
INFO[2025-06-24 19:15:52] crowdsec_wizard: service 'ssh': /var/log/auth.log
INFO[2025-06-24 19:15:52] crowdsec_wizard: using journald for 'smb'
INFO[2025-06-24 19:15:53] crowdsec_wizard: service 'linux': /var/log/syslog /var/log/kern.log
Machine '8dad094e9e5c4f93a6263bcb72896cc4sEwpyNXXWE2D4iUI' successfully added to the local API.
API credentials written to '/etc/crowdsec/local_api_credentials.yaml'.
Updating hub
Downloading /etc/crowdsec/hub/.index.json
Action plan:
🔄 check & update data files

INFO[2025-06-24 19:15:59] crowdsec_wizard: Installing collection 'crowdsecurity/apache2'
downloading parsers:crowdsecurity/apache2-logs
downloading parsers:crowdsecurity/http-logs
downloading scenarios:crowdsecurity/http-crawl-non_statics
downloading scenarios:crowdsecurity/http-probing
downloading scenarios:crowdsecurity/http-bad-user-agent
downloading https://hub-data.crowdsec.net/web/bad_user_agents.regex.txt
downloading scenarios:crowdsecurity/http-path-traversal-probing
downloading https://hub-data.crowdsec.net/web/path_traversal.txt
downloading scenarios:crowdsecurity/http-sensitive-files
downloading https://hub-data.crowdsec.net/web/sensitive_data.txt
downloading scenarios:crowdsecurity/http-sqli-probing
downloading https://hub-data.crowdsec.net/web/sqli_probe_patterns.txt
downloading scenarios:crowdsecurity/http-xss-probing
downloading https://hub-data.crowdsec.net/web/xss_probe_patterns.txt
downloading scenarios:crowdsecurity/http-backdoors-attempts
downloading https://hub-data.crowdsec.net/web/backdoors.txt
downloading scenarios:ltsich/http-w00tw00t
downloading scenarios:crowdsecurity/http-generic-bf
downloading scenarios:crowdsecurity/http-open-proxy
downloading scenarios:crowdsecurity/http-admin-interface-probing
downloading https://hub-data.crowdsec.net/web/admin_interfaces.txt
downloading scenarios:crowdsecurity/http-wordpress-scan
downloading scenarios:crowdsecurity/http-cve-probing
downloading https://hub-data.crowdsec.net/web/trendy_cves_uris.json
downloading scenarios:crowdsecurity/http-sap-interface-probing
downloading scenarios:crowdsecurity/http-generic-test
downloading contexts:crowdsecurity/http_base
downloading scenarios:crowdsecurity/http-cve-2021-41773
downloading scenarios:crowdsecurity/http-cve-2021-42013
downloading scenarios:crowdsecurity/grafana-cve-2021-43798
downloading scenarios:crowdsecurity/vmware-vcenter-vmsa-2021-0027
downloading scenarios:crowdsecurity/fortinet-cve-2018-13379
downloading scenarios:crowdsecurity/pulse-secure-sslvpn-cve-2019-11510
downloading scenarios:crowdsecurity/f5-big-ip-cve-2020-5902
downloading scenarios:crowdsecurity/thinkphp-cve-2018-20062
downloading https://hub-data.crowdsec.net/web/thinkphp_cve_2018-20062.txt
downloading scenarios:crowdsecurity/apache_log4j2_cve-2021-44228
downloading https://hub-data.crowdsec.net/web/log4j2_cve_2021_44228.txt
downloading scenarios:crowdsecurity/jira_cve-2021-26086
downloading https://hub-data.crowdsec.net/web/jira_cve_2021-26086.txt
downloading scenarios:crowdsecurity/spring4shell_cve-2022-22965
downloading scenarios:crowdsecurity/vmware-cve-2022-22954
downloading scenarios:crowdsecurity/CVE-2022-37042
downloading scenarios:crowdsecurity/CVE-2022-41082
downloading scenarios:crowdsecurity/CVE-2022-35914
downloading scenarios:crowdsecurity/CVE-2022-40684
downloading scenarios:crowdsecurity/CVE-2022-26134
downloading scenarios:crowdsecurity/CVE-2022-42889
downloading scenarios:crowdsecurity/CVE-2022-41697
downloading scenarios:crowdsecurity/CVE-2022-46169
downloading scenarios:crowdsecurity/CVE-2022-44877
downloading scenarios:crowdsecurity/CVE-2019-18935
downloading scenarios:crowdsecurity/netgear_rce
downloading scenarios:crowdsecurity/CVE-2023-22515
downloading scenarios:crowdsecurity/CVE-2023-22518
downloading scenarios:crowdsecurity/CVE-2023-49103
downloading scenarios:crowdsecurity/CVE-2017-9841
downloading scenarios:crowdsecurity/CVE-2024-38475
downloading scenarios:crowdsecurity/CVE-2024-0012
downloading scenarios:crowdsecurity/CVE-2024-9474
downloading collections:crowdsecurity/http-cve
downloading collections:crowdsecurity/base-http-scenarios
downloading collections:crowdsecurity/apache2
enabling parsers:crowdsecurity/apache2-logs
enabling parsers:crowdsecurity/http-logs
enabling scenarios:crowdsecurity/http-crawl-non_statics
enabling scenarios:crowdsecurity/http-probing
enabling scenarios:crowdsecurity/http-bad-user-agent
enabling scenarios:crowdsecurity/http-path-traversal-probing
enabling scenarios:crowdsecurity/http-sensitive-files
enabling scenarios:crowdsecurity/http-sqli-probing
enabling scenarios:crowdsecurity/http-xss-probing
enabling scenarios:crowdsecurity/http-backdoors-attempts
enabling scenarios:ltsich/http-w00tw00t
enabling scenarios:crowdsecurity/http-generic-bf
enabling scenarios:crowdsecurity/http-open-proxy
enabling scenarios:crowdsecurity/http-admin-interface-probing
enabling scenarios:crowdsecurity/http-wordpress-scan
enabling scenarios:crowdsecurity/http-cve-probing
enabling scenarios:crowdsecurity/http-sap-interface-probing
enabling scenarios:crowdsecurity/http-generic-test
enabling contexts:crowdsecurity/http_base
enabling scenarios:crowdsecurity/http-cve-2021-41773
enabling scenarios:crowdsecurity/http-cve-2021-42013
enabling scenarios:crowdsecurity/grafana-cve-2021-43798
enabling scenarios:crowdsecurity/vmware-vcenter-vmsa-2021-0027
enabling scenarios:crowdsecurity/fortinet-cve-2018-13379
enabling scenarios:crowdsecurity/pulse-secure-sslvpn-cve-2019-11510
enabling scenarios:crowdsecurity/f5-big-ip-cve-2020-5902
enabling scenarios:crowdsecurity/thinkphp-cve-2018-20062
enabling scenarios:crowdsecurity/apache_log4j2_cve-2021-44228
enabling scenarios:crowdsecurity/jira_cve-2021-26086
enabling scenarios:crowdsecurity/spring4shell_cve-2022-22965
enabling scenarios:crowdsecurity/vmware-cve-2022-22954
enabling scenarios:crowdsecurity/CVE-2022-37042
enabling scenarios:crowdsecurity/CVE-2022-41082
enabling scenarios:crowdsecurity/CVE-2022-35914
enabling scenarios:crowdsecurity/CVE-2022-40684
enabling scenarios:crowdsecurity/CVE-2022-26134
enabling scenarios:crowdsecurity/CVE-2022-42889
enabling scenarios:crowdsecurity/CVE-2022-41697
enabling scenarios:crowdsecurity/CVE-2022-46169
enabling scenarios:crowdsecurity/CVE-2022-44877
enabling scenarios:crowdsecurity/CVE-2019-18935
enabling scenarios:crowdsecurity/netgear_rce
enabling scenarios:crowdsecurity/CVE-2023-22515
enabling scenarios:crowdsecurity/CVE-2023-22518
enabling scenarios:crowdsecurity/CVE-2023-49103
enabling scenarios:crowdsecurity/CVE-2017-9841
enabling scenarios:crowdsecurity/CVE-2024-38475
enabling scenarios:crowdsecurity/CVE-2024-0012
enabling scenarios:crowdsecurity/CVE-2024-9474
enabling collections:crowdsecurity/http-cve
enabling collections:crowdsecurity/base-http-scenarios
enabling collections:crowdsecurity/apache2

Run 'sudo systemctl reload crowdsec' for the new configuration to be effective.
INFO[2025-06-24 19:16:01] crowdsec_wizard: Installing collection 'crowdsecurity/linux'
downloading parsers:crowdsecurity/syslog-logs
downloading parsers:crowdsecurity/geoip-enrich
downloading https://hub-data.crowdsec.net/mmdb_update/GeoLite2-City.mmdb
downloading https://hub-data.crowdsec.net/mmdb_update/GeoLite2-ASN.mmdb
downloading parsers:crowdsecurity/dateparse-enrich
downloading parsers:crowdsecurity/sshd-logs
downloading scenarios:crowdsecurity/ssh-bf
downloading scenarios:crowdsecurity/ssh-slow-bf
downloading scenarios:crowdsecurity/ssh-cve-2024-6387
downloading scenarios:crowdsecurity/ssh-refused-conn
downloading scenarios:crowdsecurity/ssh-generic-test
downloading contexts:crowdsecurity/bf_base
downloading collections:crowdsecurity/sshd
downloading collections:crowdsecurity/linux
enabling parsers:crowdsecurity/syslog-logs
enabling parsers:crowdsecurity/geoip-enrich
enabling parsers:crowdsecurity/dateparse-enrich
enabling parsers:crowdsecurity/sshd-logs
enabling scenarios:crowdsecurity/ssh-bf
enabling scenarios:crowdsecurity/ssh-slow-bf
enabling scenarios:crowdsecurity/ssh-cve-2024-6387
enabling scenarios:crowdsecurity/ssh-refused-conn
enabling scenarios:crowdsecurity/ssh-generic-test
enabling contexts:crowdsecurity/bf_base
enabling collections:crowdsecurity/sshd
enabling collections:crowdsecurity/linux

Run 'sudo systemctl reload crowdsec' for the new configuration to be effective.
INFO[2025-06-24 19:16:09] crowdsec_wizard: Installing collection 'crowdsecurity/nginx'
downloading parsers:crowdsecurity/nginx-logs
downloading scenarios:crowdsecurity/nginx-req-limit-exceeded
downloading collections:crowdsecurity/nginx
enabling parsers:crowdsecurity/nginx-logs
enabling scenarios:crowdsecurity/nginx-req-limit-exceeded
enabling collections:crowdsecurity/nginx

Run 'sudo systemctl reload crowdsec' for the new configuration to be effective.
INFO[2025-06-24 19:16:09] crowdsec_wizard: Installing collection 'crowdsecurity/smb'
downloading parsers:crowdsecurity/smb-logs
downloading scenarios:crowdsecurity/smb-bf
downloading collections:crowdsecurity/smb
enabling parsers:crowdsecurity/smb-logs
enabling scenarios:crowdsecurity/smb-bf
enabling collections:crowdsecurity/smb

Run 'sudo systemctl reload crowdsec' for the new configuration to be effective.
downloading parsers:crowdsecurity/whitelists
enabling parsers:crowdsecurity/whitelists

Run 'sudo systemctl reload crowdsec' for the new configuration to be effective.
Created symlink /etc/systemd/system/multi-user.target.wants/crowdsec.service → /lib/systemd/system/crowdsec.service.
Not attempting to start crowdsec, port 8080 is already used or lapi was disabled
This port is configured through /etc/crowdsec/config.yaml and /etc/crowdsec/local_api_credentials.yaml
Get started with CrowdSec:
 * Go further by following our post installation steps : https://docs.crowdsec.net/u/getting_started/next_steps
====================================================================================================================
 * Install a remediation component to block attackers: https://docs.crowdsec.net/u/bouncers/intro
====================================================================================================================
 * Find more collections, parsers and scenarios created by the community with the Hub: https://hub.crowdsec.net
====================================================================================================================
 * Subscribe to additional blocklists, visualize your alerts and more with the console: https://app.crowdsec.net
```

You can always run the configuration again interactively by using '/usr/share/crowdsec/wizard.sh -c'

```sh
sudo cscli alerts list
sudo cscli decisions list
cscli metrics
```

```bash
sudo cscli collections list
```

**Configure CrowdSec:**

- Edit the `acquis.yaml` file to configure log sources: 

```
sudo nano /etc/crowdsec/acquis.yaml
```

- Install relevant collections, such as for SSH: 

```
sudo cscli collections install crowdsecurity/sshd
```

And configure the corresponding log file:

```
sudo nano /etc/crowdsec/acquis.d/sshd.yaml
```

1. **Restart CrowdSec:**

```
sudo systemctl restart crowdsec
```

https://docs.crowdsec.net/docs/v1.4.0/getting_started/install_crowdsec/
https://doc.crowdsec.net/index.html