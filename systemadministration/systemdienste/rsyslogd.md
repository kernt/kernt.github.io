https://ackcent.com/basics-linux-events-logging/

In this post we will cover the basics of Event Logging in Linux systems. We will talk about Syslog: Message structure, the most famous implementations and main configurations. We will also play with the inner workings of Linux logging using UNIX sockets, logger and syslog services.

Let’s get to it!

## What is Syslog?

Syslog is a standard (RFC5424) used for log management. This management can be local or remote. Do not confuse syslog standard with syslog applications like Syslog-ng, Rsyslog, Nxlog”¦

In some of the most famous Linux distros like Ubuntu, Debian or Fedora, Rsyslog is installed by default. This application can be used for local or remote log management.

## Message structure

Syslog message structure is similar to this:

- TIMESTAMP HOSTNAME SERVICE [PID]: MESSAGE

Example:

- <34>Oct 11 22:14:15 mymachine su: ‘su root’ failed for riff on /dev/pts/8

PRI value is calculated using the facility and priority values (see tables and formula below). Facility and Severity values are not normative but often used. They are described in the following tables for purely informational purposes. Facility values MUST be in the range of 0 to 23 inclusive.

![facility 1](https://www-wordpress-thevoo6roh7taita.ackcent.com/wp-content/uploads/2019/04/facility-1.png)

![security level1](https://www-wordpress-thevoo6roh7taita.ackcent.com/wp-content/uploads/2019/04/security-level1.png)

The formula we will use to calculate the PRI is:

- **Priority=Facility*8+Severity**

This result will be between 0 and 191.
## Log storage

We already know the standard syslog message structure. Now, how are these messages managed?

In Linux environments we have a special type of files called “UNIX sockets”. These files are used to transfer information between processes. Processes will read and write data to the socket in order to communicate. In Debian we can list UNIX sockets with the following command:


In our case, applications will write their log messages into this Syslog dedicated socket. Rsyslog service (in Debian/Ubuntu/Fedora) will be “listening” (reading) from this socket. Every new log entry will then be processed according to Rsyslog configuration (later mentioned).
We can send a random message to this socket in order to test our Rsyslog configuration:
Note that you can specify a PRI with the following command: “logger -p kern.err hola”

## Rsyslog service

As we have just seen, Rsyslog must be always listening to any log income from syslog UNIX socket. In order to achieve this, Rsyslog service must be always up and running (obviously). You can check its status with the following command:

- **Systemctl status Rsyslog.service**

One of our main questions is: Which criteria does Rsyslog use to classify and store logs in different files under /var/log? Can this location be modified?

Of course. Rsyslog behaviour is defined in its configuration file and can be easily modified:

- **/etc/Rsyslog.d/50-default.conf**

This is some of its contents by default:

![3](https://www-wordpress-thevoo6roh7taita.ackcent.com/wp-content/uploads/2020/05/3.jpeg)

The storage rules are defined in the first lines following this syntax:

- **Facility.severity storage_path**

For example, by defining:

- **cron.* /var/log/cron.log**

All cron facility logs of any severity will be stored under /var/log/cron.log

If we want to send these logs to a remote machine (usually a Log Collector) we will use the following settings (depending on if we want to use TCP or UDP):

- **UDP: cron.* @192.168.1.30:514**
- **TCP: cron.* @@192.168.1.30:514**

In general, it is suggested to use TCP syslog since it is way more reliable than UDP. Keep in mind that in case of network congestion, messages might be dropped. We UDP these logs will be lost, but with TCP they will be resend after the network issue. Also note that a TLS option is available in order to send logs, but only with TCP. Nevertheless, TCP is not fully reliable and might also introduce some losses [(see this post about TCP reliability)](https://rainer.gerhards.net/2008/04/on-unreliability-of-plain-tcp-syslog.html). In order to increase reliability, see the RELP protocol from Rsyslog (See below) or the standard RFC3195.

In order to use it, install rsyslog-relp in both client and server (collector). In Debian:

- **sudo apt install rsyslog-relp**

Then you should add a couple of lines in the config files:

In client:

- **module(load=”omrelp”)**
- ***.* :omrelp:192.168.1.1:4444**

In server:

- **module(load=”imrelp”)**
- **input(type=”imrelp” port=”4444″ maxDataSize=”10k”)**

Then make sure you restart Rsyslog service in both machines.

- **sudo systemctl restart rsyslog**

RELP is not part of standard yet and has only been implemented in the following applications:

- librelp – the original C RELP library
- rsyslog (Windows and Linux)
- MonitorWare (Windows)
- logstash
## Standard locations under /var/log/

Even though the destination files are fully configurable -as we have just seen-, we will try to maintain some standard names and log files under our /var/log directory:

- **Messages (or syslog)**: General messages. Useful for a first look at logs
- **Kern.log**: Kernel logs
- **Auth.log (or secure)**: Authorization logs, users’ sessions
- **Dmesg**: Device driver messages. Use “dmesg” to see its content
- **Faillog**: Log of failed logins
- **Cron**: Info related to cron jobs
- **Daemon.log**: Info about background services (daemons)
- **Btmp**: Failed login attempts
- **Utmp**: Current login state by user
- **Wtmp**: Record of each login/logout
- **Lastlog**: Record of the last login
- **Yum/apt**: Logs of package installations
- **User.log**: Info about all user level logs (coming from user programs)
- **Xorg.x.log**: Logs about the X (graphic interface)
## Log rotation with Rsyslog

Log rotation is the action of saving (usually compressed) old logs for a determined amount of time. In Debian we can achieve this with “logrotate”. This is one of its configuration files (under /etc/logrotate.d/), for Rsyslog it is pretty self-explanatory:
### RFC 3164 vs. RFC 5424

While RFC 5424 is the current Syslog protocol, it’s not the only standard you’ll see in the wild. RFC 3164 (a.k.a. “BSD syslog” or “old syslog”) is an older syslog format still used by many devices. In practice, admins are likely to see syslog messages that use both RFC 3164 and RFC 5424 formatting.

Good indicators of an RFC 3164 syslog message are the absence of structured data and timestamps using an “Mmm dd hh:mm:ss” format.

Here are some examples of what BSD messages look like, using [section 5.4 of RFC 3164](https://datatracker.ietf.org/doc/html/rfc3164#section-5.4) as a reference:

`<34>Nov 11 11:11:11 pepeggserver su: 'su admin' failed **for** user1 **on** /dev/**pts**/0`

`<13>**Nov**  11 11:11:11 198.51.100.11 **Read** **the** **docs**!`

We’ll focus on the newer RFC 5424 protocol here, but keep RFC 3164 in mind if you see messages that don’t conform to RFC 5424.

## Syslog Format and Fields

Syslog messages contain a [standardized header](https://tools.ietf.org/html/rfc5424#section-6) with several fields. These include the timestamp, the name of the application that generated the event, the location in the system where the message originated, and its priority. You can change this format in your syslog implementation’s configuration file, but using the standard format makes it easier to parse, [analyze](https://www.loggly.com/ultimate-guide/analyzing-linux-logs/), and route syslog events.

Here’s an example log message using the default format. It’s from the SSH daemon (sshd), which controls remote logins to the system. This message describes a failed login attempt:

`Jun 4 22:14:15 server1 sshd[41458] : Failed password for root from 10.0.2.2 port 22 ssh2`

You can also add additional fields to your syslog messages. Let’s repeat the last event after adding a few new fields. We’ll use the following rsyslog template, which adds the priority (<%pri%>), protocol version (%protocol-version%), and the date formatted using [RFC 3339](https://tools.ietf.org/html/rfc3339) (%timestamp:::date-rfc3339%):

`<%pri%>%protocol-version% %timestamp:::date-rfc3339% %HOSTNAME% %app-name% %procid% %msgid% %msg%n`

This generates the following log:

`<34>1 2019-06-05T22:14:15.003Z server1 sshd - - pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=10.0.2.2`

Below, you’ll find descriptions of some of the most commonly used syslog fields when searching or [troubleshooting issues](https://www.loggly.com/ultimate-guide/troubleshooting-with-linux-logs/).

### [Priority](https://tools.ietf.org/html/rfc5424#section-6.2.1)

The priority field or [pri](https://tools.ietf.org/html/rfc5424#section-6.2.1) for short (“<34>” in the example) tells you how urgent or severe the event is. It’s a combination of two numerical fields: the facility and the severity. The facility specifies the type of process that created the event, ranging from 0 for kernel messages to 23 for local applications. The severity ranges from 0 – 7, with 0 indicating an emergency and 7 indicating a debug event.

Pri can be output in two ways. The first is as a single number, prival, which is calculated as the facility field value multiplied by eight; then, the result is added to the severity field value: (facility)(8) + (severity). The second is pri-text, which will output in the string format “facility.severity”. The latter format is often easier to read and search but takes up more storage space.

### Timestamp

The [timestamp](https://tools.ietf.org/html/rfc5424#section-6.2.3) field (“2019-06-05T22:14:15.003Z” in the above example) indicates the time and date the message was generated on the system sending the message. The example timestamp breaks down like this:

- “2019-06-05” is the year, month, and day.
- “T” is a required element of the timestamp field, separating the date and the time.
- “22:14:15.003” is the 24-hour format of the time, including the number of milliseconds (003).
- “Z” indicates UTC time. Instead of Z, the example could have included an offset, such as -08:00, which indicates that the time is offset from UTC by eight hours.
### [Hostname](https://tools.ietf.org/html/rfc5424#section-6.2.4)

The [hostname](https://tools.ietf.org/html/rfc5424#section-6.2.4) field (“server1” in the example) indicates the name of the host or system that originally sent the message.
### [App-name](https://tools.ietf.org/html/rfc5424#section-6.2.5)

The [app-name](https://tools.ietf.org/html/rfc5424#section-6.2.5) field (“sshd:auth in the example) indicates the name of the application that sent the message.
## Logging with systemd

Many Linux distributions ship with systemd—a process and service manager. Systemd implements its own logging service called journald, which can replace or complement Syslog. Journald logs in a significantly different manner than systemd, which is why it has [its own section in the Ultimate Guide to Logging](https://www.loggly.com/ultimate-guide/using-journalctl/). You can learn more about logging via systemd in the [Systemd Logging](https://www.loggly.com/ultimate-guide/category/systemd/) section.