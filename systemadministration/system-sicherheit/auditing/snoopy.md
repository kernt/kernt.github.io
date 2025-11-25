## Installing Snoopy

### Debian / Ubuntu

`apt install snoopy`

During installation it will ask your permission to add the wrapper to **/etc/ld.so.preload**, so it can be executed and act as a middle-man.

If the library is listed, new commands should be “intercepted” and logged to your auth.log.

`tail /var/log/auth.log`

The installation of Snoopy is easy and quick. No further configuration is needed at this point, although you might want to consider to configure remote syslog. This way the log (and audit trail) is stored on an external location

