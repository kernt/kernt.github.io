---
tags:
  - monitoring
  - alertmanager
  - prometheus
  - amtool
---
# amtoll Konfiguration

[](https://github.com/prometheus/alertmanager#configuration)

`amtool` allows a configuration file to specify some options for convenience. The default configuration file paths are `$HOME/.config/amtool/config.yml` or `/etc/amtool/config.yml`

An example configuration file might look like the following:

```
# Define the path that `amtool` can find your `alertmanager` instance
alertmanager.url: "http://localhost:9093"

# Override the default author. (unset defaults to your username)
author: me@example.com

# Force amtool to give you an error if you don't include a comment on a silence
comment_required: true

# Set a default output format. (unset defaults to simple)
output: extended

# Set a default receiver
receiver: team-X-pager
```

# amtool routen

`amtool` allows you to visualize the routes of your configuration in form of text tree view. Also you can use it to test the routing by passing it label set of an alert and it prints out all receivers the alert would match ordered and separated by `,`. (If you use `--verify.receivers` amtool returns error code 1 on mismatch)

Example of usage:

```
# View routing tree of remote Alertmanager
$ amtool config routes --alertmanager.url=http://localhost:9090

# Test if alert matches expected receiver
$ amtool config routes test --config.file=doc/examples/simple.yml --tree --verify.receivers=team-X-pager service=database owner=team-X
```

# amtool Benutzen

 Der alertmanager tool wird read eine Konfiguration im YAML format from von einer der beiden default locations: 
 
 *$HOME/.config/amtool/config.yml* 
 oder
*/etc/amtool/config.yml*

Create a config file for

```sh
sudo mkdir -p /etc/amtool
```

Enter the following content in the config file

```sh
alertmanager.url: http://localhost:9093
```

Verify `amtool` is working by pulling the current Alert-manager configuration

```sh
amtool config show
```

**Alle arlamierungen ausgeben**

```sh
amtool -o extended alert
```

**Eine Alarm unterdrücken**

```
amtool silence add alertname=Test_Alert
amtool silence add alertname="Test_Alert" instance=~".+0"
```

**Unterdrückte Arlame ausgeben**

```
amtool silence query
amtool silence query instance=~".+0"
```

Expire a silence:

```
$ amtool silence expire b3ede22e-ca14-4aa0-932c-ca2f3445f926
```

Expire all silences matching a query:

```
$ amtool silence query instance=~".+0"
ID                                    Matchers                            Ends At                  Created By  Comment
e48cb58a-0b17-49ba-b734-3585139b1d25  alertname=Test_Alert instance=~.+0  2017-08-02 22:41:39 UTC  kellel

$ amtool silence expire $(amtool silence query -q instance=~".+0")

$ amtool silence query instance=~".+0"

```

Expire all silences:

```
$ amtool silence expire $(amtool silence query -q)
```

Try out how a template works. Let's say you have this in your configuration file:

```
templates:
  - '/foo/bar/*.tmpl'
```

Then you can test out how a template would look like with example by using this command:

```
amtool template render --template.glob='/foo/bar/*.tmpl' --template.text='{{ template "slack.default.markdown.v1" . }}'
```