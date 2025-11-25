
| Netzname |         Netzaddresse         |     Infos      |
| :------: | :--------------------------: | :------------: |
| docker0  |        172.17.0.1/16         |  Docker netz   |
|          |        192.168.4.0/21        |    homenet     |
|          |       192.168.200.0/23       | wireguard vpn  |
|          |        10.233.64.0/18        | calico ip pool |
|          | fd85:ee78:d8a6:8607::1:0/112 | calico ip pool |
# Core DNS

10.233.3
10.233.114.65
10.233.64.0/18
# Contabo DNS Server

213.136.95.10
213.136.95.11
2a02:C207::1:53
# Kube DNS

169.254.25.10



