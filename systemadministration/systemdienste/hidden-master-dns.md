---
tags:
  - bind
  - named
---
# DNS Bind hidden master

## Configuring my BIND/named DNS servers to operate from a hidden master via VPN for Let’s Encrypt

*named.conf*

```c
notify explicit;
allow-transfer { "none"; };
listen-on      { 127.0.0.1; 10.15.0.53; };
```

**Der nsupdate key**

```c
key me.example.org {
    algorithm HMAC-SHA512;
    secret "TgDiZn2IpmHjlF2B4qaHWHuBH3G4G76weISdPbt9iSNn8o9H5/yTDBNpJvw4y6wNz8+R5BpZ9r/wqnzeMphW8Q==";
};
```

**Die slaves**

```c
// this is usually kept in sync with acl SlaveServersVPN
// these IP addresses are those upon which named listens
masters SlaveServersToNotify {
        10.180.0.85;
        10.170.0.68;
        10.130.0.66;
};
```

**Woher kommen Zonenübertragungen?**

```c
// this is usually kept in sync with masters SlaveServersToNotify
// these IP address are often the VPN end-points.
// These are the IPs from which zone transfers will originate.
acl SlaveServersVPN {
        10.180.0.85;
        10.170.0.68;
        10.18.1.30;
};
```

**Wer kann eine Übertragung veranlassen?**

```c
acl AllowZoneTransfer {
        LocalTrustedBoxes;
        SlaveServersVPN;
};
```

**Die Zonendeklaration auf dem Master**

```c
zone "langille.org" {
        type master;
        file "zones/langille.org.db";
        allow-transfer { AllowZoneTransfer; };
        update-policy  { grant me.example.org zonesub TXT; };
        notify yes;
        also-notify    { SlaveServersToNotify; };
};
```

**Die Slave-Server-Konfiguration**

Auf meinen Slave-Servern möchte ich nicht, dass sie Benachrichtigungen versenden, sofern nicht anders angegeben, daher habe ich dies zur Optionsklausel in der Datei „named.conf“ hinzugefügt

`notify explicit;`

Ich möchte auch, dass dieser Slave über das VPN erreichbar ist, also habe ich dies zur gleichen Optionsklausel hinzugefügt

`listen-on   { 127.0.0.1; 10.180.0.85; [REDACTED PUBLIC IP ADDRESS]; };`

Die Zone und die damit verbundenen Details werden auf diese Weise im Slave angegeben

```c
masters HiddenMaster { 10.15.0.53; };

acl AllowZoneTransfer {
        [REDACTED LIST OF IP ADDRESSES];
};

zone "langille.org" {
        type slave;
        file "secondary/langille.org.db";
        masters { HiddenMaster; };
        allow-transfer { AllowZoneTransfer; };
        notify no;
};
```
Zeile 1 identifiziert den zuvor in diesem Artikel beschriebenen versteckten Master

Zeile 3 Gibt an, wer von diesem Slave aus eine Zonenübertragung durchführen kann. Auf diese Weise kann ich von ausgewählten Hosts aus testen, was sich auf diesem bestimmten Slave befindet.

Zeile 7 definiert die Zone und Zeile 8 gibt an, dass es sich um eine Slave-Zone handelt.

Zeile 10 listet die Meister auf.

Zeile 11 sagt wer bei uns einen Zonentransfer machen kann.

Zeile 12 sagt, dass die für diese Zone aufgeführten NS-Server nicht benachrichtigt werden sollen, wenn eine Zonenaktualisierung erfolgt

## DNS Hosting Guide: Hidden Master with DNSSEC

*named.conf*

```c
// IP addresses of your secondary nameservers, where the zone transfers will
// take place. Your DNS provider should give you these addresses.
// NOTE - these IPs are probably not the same as your DNS provider's public
// nameservers (which will go in your zone file's ns1-3 A records).
masters secondaries {
  198.51.100.51;
  198.51.100.52;
  198.51.100.53;
};

// same as above - repetition needed for BIND's arcane config syntax
acl secondaries {
  198.51.100.51;
  198.51.100.52;
  198.51.100.53;
};

// localhost, and any other public IPs of your server
acl localnetworks {
  127.0.0.1;
  ::1;
  203.0.113.41;
  203.0.113.42;
  2001:db8::2;
  2001:db8::3;
};

options {
  directory       "/usr/local/etc/namedb/working";
  pid-file        "/var/run/named/pid";
  dump-file       "/var/dump/named_dump.db";
  statistics-file "/var/stats/named.stats";
  key-directory   "/usr/local/etc/namedb/keys";

  // listen on all interfaces (needed for zone transfers to secondaries)
  listen-on    { any; };
  listen-on-v6 { any; };

  // IP addresses of your upstream DNS servers. Your hosting provider most
  // likely provides a DNS server. Alternatively, leave blank to have BIND
  // recursively resolve all queries (slow).
  //
  // Note - below are Google's DNS servers. If you value your privacy, don't
  // use them.
  forwarders {
    8.8.8.8;
    8.8.4.4;
    2001:4860:4860::8888;
    2001:4860:4860::8844;
  };

  // If a lookup fails on upstream DNS servers, don't try to recursively
  // resolve (comment out if not using forwarders).
  forward only;

  // When a zone is updated, only send NOTIFY to hosts in the zone's
  // `also-notify` block (defined below).
  notify explicit;

  // Set safe default permissions. We will override these for some zones
  // below.
  allow-transfer  { none; };
  allow-update    { none; };
  allow-recursion { localnetworks; };
  allow-query     { localnetworks; };

  // If your server has multiple IP addresses, uncomment the two lines below
  // and change the source address to the one you configured your secondary
  // DNS provider to use.
  //  query-source address 203.0.113.41;
  //  notify-source        203.0.113.41;

  // Comment out the three lines below if you don't want DNSSEC.
  dnssec-enable yes;
  dnssec-validation auto;
  dnssec-lookaside auto;
};

// Don't attempt to resolve any private IPs
include "/usr/local/etc/namedb/localzones.conf";

// Your domain name goes here.
zone "example.com" in {
  // We are the "master" (or primary) server for our domain, but it just
  // happens that we won't be getting any external queries.
  type master;

  // Comment out the two lines below if you don't want DNSSEC.
  auto-dnssec maintain;
  inline-signing yes;

  // only allow zone transfers from localhost or our secondary DNS provider
  allow-transfer {
    localnetworks;
    secondaries;
  };

  // only allow DNS queries from localhost or our secondary DNS provider
  allow-query {
    localnetworks;
    secondaries;
  };

  // send NOTIFY messages to secondary DNS provider when the zone changes
  also-notify {
    secondaries;
  };

  // your domain's zone file goes here
  file "/usr/local/etc/namedb/master/example.com.db";
};
```

## DNSSEC Einrichten

First, generate a Zone Signing Key (ZSK) for your domain by running the following

```sh
dnssec-keygen -a ECDSAP256SHA256 -K /usr/local/etc/namedb/keys -n ZONE example.com
Generating key pair.
Kexample.com.+013+29679
```

This will generate a keypair for the example.com in BIND's key directory. Next, generate a Key Signing Key (KSK) for the domain in a similar fashion:

```sh
dnssec-keygen -f KSK -a ECDSAP256SHA256 -K /usr/local/etc/namedb/keys -n ZONE example.com
Generating key pair.
Kexample.com.+013+15315
```

Take note of the KSK's generated filename—you'll need it in a bit. You should now have 4 files for this domain in BIND's key directory: one pair for the ZSK, and another pair for the KSK. We will come back to these at the end of the guide when you configure DNS settings at your registrar.

Writing a Zone File

../master//example.com.db

```
; The $TTL variable defines a default TTL for all records in this file.
; This value specifies how long other DNS servers should keep a record in
; their cache. Individual records may override this value.
$TTL 3h

; your bare domain name
$ORIGIN example.com.

; "Start of Authority" record. First line should contain your  primary
; nameserver and administrative contact's email (replace '@' with '.')
@  IN  SOA  ns1.example.com.  root.example.com. (

; serial:   you *must* increment this number whenever any change is made
;           to this file, otherwise updates will not propagate to your
;           secondary DNS servers
2017101800
; refresh:  how often your secondary DNS servers should poll your primary
;           server for changes
1d
; retry:    how long your secondary DNS servers should wait before
;           retrying after a failed update
3m
; expire:   how long your secondary DNS servers should be considered
;           authoritative if your primary nameserver disappears
1w
; minimum:  "negative caching TTL," or how long other servers should wait
;           before re-querying a record that didn't exist on the previous
;           attempt
3h
)

; Your domain's public nameservers go in the NS records here. You'll need
; an A record for each one that points to your secondary DNS provider.
IN  NS     ns1.example.com.
IN  NS     ns2.example.com.
IN  NS     ns3.example.com.

; MX record (if you have a mail server)
IN  MX  10 mail.example.com.

; server host definitions
@           IN  A      203.0.113.41
@           IN  AAAA   2001:db8::2
mail        IN  A      203.0.113.42
mail        IN  AAAA   2001:db8::3
awesomebox  IN  A      203.0.113.43
awesomebox  IN  AAAA   2001:db8::4

; These records should contain the IP addresses of your secondary DNS
; provider's PUBLIC nameservers. Clients will use these DNS servers when
; querying your domain.
ns1         IN  A      198.51.100.11
ns1         IN  AAAA   2001:db8:beef::7
ns2         IN  A      198.51.100.12
ns2         IN  AAAA   2001:db8:beef::8
ns3         IN  A      198.51.100.13
ns3         IN  AAAA   2001:db8:beef::9
```

**Configuring Zone Transfers**

Check /var/log/messages to ensure BIND started up properly. Hopefully you didn't make any typos. You can verify BIND is working by doing a simple DNS query

```
dig @127.0.0.1 +short google.com
172.217.5.206
```

**Also, make sure BIND is correctly serving the domain you configured in your zone file:**

```
dig @127.0.0.1 +short example.com
203.0.113.41
```

**Notifying Your Registrar**

Your registrar has the secret sauce that tells other DNS servers which nameservers to query for information about your domain. Once you've verified everything is working, you can switch your nameservers over to your secondary DNS provider. You probably want to do this late at night, or during a time when you don't expect much traffic to your site. I use Namecheap, but the instructions should carry over to most other registrars.

In your registrar's web portal, your domain's settings page should have an option to set your nameservers. At Namecheap, you want to select Custom DNS. You will then need to provide the fully qualified domain name of your secondary DNS provider's public DNS servers. As DNS Made Easy states at the top of their portal: These are the name servers that you will want to add as NS records in your zone and also assign to your domain at your domain registrar.
Side Note: Vanity Nameservers

If you'd like your public nameservers for example.com to look like ns1.example.com instead of ns1.yourdnsprovider.com, then you can configure "vanity nameservers" for your domain. First, make sure you have A records (and probably AAAA records) for your secondary DNS provider's public name servers in your zone file (I emphasized this in the example zone file above).

I can't speak for other registrars, but at Namecheap, you can go to the Advanced DNS page for your domain and scroll down to Personal DNS Server. Then you can add ns1, ns2... and map them to the IP addresses of your secondary DNS provider's public nameservers.

Then, in the Custom DNS field, you can just put ns1.example.com, ns2.example.com, etc.

Doing this creates a "glue record" at your registrar for your domain's DNS servers. You can Google this if you're interested in the details. Also, the web interface at Namecheap currently only allows IPv4 glue records. I had to open a support ticket to have IPv6 glue records added for DNS Made Easy's public IPv6 servers.
Optional: DNSSEC at the Registrar

If you configured DNSSEC above, there is one last step to complete while you're on your registrar's web portal. You'll need to provide the SHA-1 and SHA-256 digests of the KSK you generated to your registrar. Your registrar will then add some fancy records for you so that other resolvers can cryptographically verify your DNS records. (Sorry for the hand-wavy explanation—it's 1:00 AM on a Friday night and I'm tired.)

Remember the filename I asked you to remember from the dnssec-keygen command above? We'll use it now. You want the file containing the Key Signing Key (not the Zone Signing Key). It's annoying, because the filenames look exactly the same except for a four-digit identifier at the end. It's easy to figure out though—the contents of the .key file will tell you which one it is.

Once you've found the correct file, run the following command:

```sh
dnssec-dsfromkey /usr/local/etc/namedb/keys/Kexample.com.+013+15315.key | awk '{print $4, $5, $6, $7}'
15315 13 1 7F5ED13547D9860965437D0F7CB6BFA7C70F1F62
15315 13 2 AD0C012F749F0422E0CC88D11B65248E78945F149572D54C8AE7C5B63C30E621
```

You will use this output to populate the DS records for your domain at your registrar. At Namecheap, this is located under Advanced DNS→DNSSEC. The first column is the key tag, which corresponds to the key's file name. The second column is the encryption type. If you used ECDSA, this will be 13. If you went with RSA, you'll use 8 here. The third column is the digest type: 1 for SHA-1 and 2 for SHA-256. The final column contains the actual digest.

These columns correspond to the four fields for DS records in the Namecheap web portal. Copy and paste the appropriate values there. Other registrars should have a similar process.

It will take an hour or so for your DNS updates to propagate. You can verify DNSSEC is working by by querying a different DNS server (the below example uses Google's public DNS).

```sh
dig @8.8.8.8 example.com +dnssec | grep -m1 flags:
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1
```

If you see the ad flag, then your DNSSEC was validated successfully. You can also check your DNSSEC status on [Verisign's test page](https://dnssec-debugger.verisignlabs.com/).

**Sources**

* [hidden-master-via-vpn-for-lets-encrypt](https://dan.langille.org/2017/07/01/configuring-my-bindnamed-dns-servers-to-operate-from-a-hidden-master-via-vpn-for-lets-encrypt/)
* [dns-hidden-master](https://www.c0ffee.net/blog/dns-hidden-master/)
* [namecheap](https://www.namecheap.com/?aff=108349)
