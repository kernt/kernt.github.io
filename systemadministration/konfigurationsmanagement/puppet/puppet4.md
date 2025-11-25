---
tags:
  - puppet
  - konfigurationsmanagement
---
# Puppet4

## SSL connect Fehler certificate verify failed

Meldung:

```sh
 gem update --system
ERROR:  While executing gem ... (Gem==RemoteFetcher==FetchError)
    SSL_connect returned=1 errno=0 state=error: certificate verify failed (http../puppet//api.rubygems.o../puppet/specs.4.8.gz)

```

Fehler kommt z.B bei dem versuch puppet zu installieren

* Puppet version 4.10

Download und Entpacken der Aktuellen version von rubygem

```sh
wget http../puppet//rubygems.o../puppet/rubyge../puppet/rubygems-2.6.11.tgz $$ \
tar -xzvf rubygems-2.6.11.tgz && \
cd  rubygems-2.6.11
ruby../puppet/setup.rb
```

Ausgabe der Installation

```sh
gem install puppet
Fetching: facter-2.4.6.gem (100%)
Successfully installed facter-2.4.6
Fetching: hiera-3.3.1.gem (100%)
Successfully installed hiera-3.3.1
Fetching: json_pure-1.8.6.gem (100%)
Successfully installed json_pure-1.8.6
Fetching: fast_gettext-1.1.0.gem (100%)
Successfully installed fast_gettext-1.1.0
Fetching: locale-2.1.2.gem (100%)
Successfully installed locale-2.1.2
Fetching: text-1.3.1.gem (100%)
Successfully installed text-1.3.1
Fetching: gettext-3.2.2.gem (100%)
Successfully installed gettext-3.2.2
Fetching: gettext-setup-0.24.gem (100%)
Successfully installed gettext-setup-0.24
Fetching: puppet-4.10.0.gem (100%)
Successfully installed puppet-4.10.0
...
```
