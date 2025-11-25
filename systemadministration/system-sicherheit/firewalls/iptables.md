---
tags:
  - sicherheit
  - iptables
---
# iptables Short Cuts

**iptables Reseten**

`iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X`

**iptabels block ip**

`iptables -A INPUT -s 64.39.102.147 -j DROP`
