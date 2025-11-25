---
tags:
  - sicherheit
  - firewall
  - pfsense
---
[PfSense](https://docs.netgate.com/pfsense/en/latest/development/develop-packages.html)

# Packete entwickeln

Bestandteile des Packet Systems

- Eine Manifests Datei
- Packet konfigurations Datein
- Unterstützungs Dateien (.inc Files, für web Interfaces noch zuzüglich .php Dateien)
## Die Manifests  Datei

Wo ist das Verzeichnis in dem die Manifest Dateien zu finden sind:
<category>/pfSense-pkg-<package name>/files/usr/local/share/pfSense-pkg-<package name>/info.xml

Hier eine sysutils/pfSense-pkg-Cron/files/usr/local/share/pfSense-pkg-Cron/info.xml Beispeldatei.

```xml
<pfsensepkgs>
    <package>
        <name>someprogram</name>
        <descr><![CDATA[Some cool program.]]></descr>
        <version>%%PKGVERSION%%</version>
        <configurationfile>someprogram.xml</configurationfile>
    </package>
</pfsensepkgs>
```

## Packet Konfigurationsdateien

Example:
```XML 
<?xml version="1.0" encoding="utf-8" ?>
<packagegui>
  <copyright></copyright>
  <name></name>
  <title></title>
  <include_file></include_file>
  <aftersaveredirect></aftersaveredirect>
  <menu>
    <name></name>
    <section></section>
    <configfile></configfile>
    <tooltiptext></tooltiptext>
    <url>/pkg.php?xml=package.xml</url>
  </menu>
  <tabs>
    <tab>
      <text></text>
      <url></url>
      <active/>
      <tab_level/>
    </tab>
  </tabs>
  <service>
    <name></name>
    <rcfile></rcfile>
    <executable></executable>
    <description></description>
  </service>
  <plugins>
    <item>
      <type>plugin_name</type>
    <item>
  </plugins>
  <adddeleteeditpagefields>
    <columnitem>
      <fielddescr></fielddescr>
      <fieldname></fieldname>
    </columnitem>
  </adddeleteeditpagefields>
  <fields>
    <field>
      <fielddescr></fielddescr>
      <fieldname></fieldname>
      <description></description>
      <size></size>
      <type></type>
    </field>
  </fields>
  <custom_php_global_functions><!-- PHP function call --></custom_php_global_functions>
  <custom_php_install_command><!-- PHP function call --></custom_php_install_command>
  <custom_php_pre_deinstall_command><!-- PHP function call --></custom_php_pre_deinstall_command>
  <custom_php_deinstall_command><!-- PHP function call --></custom_php_deinstall_command>
  <custom_add_php_command><!-- PHP function call --></custom_add_php_command>
  <custom_add_php_command_late><!-- PHP function call --></custom_add_php_command_late>
  <custom_delete_php_command><!-- PHP function call --></custom_delete_php_command>
  <custom_php_resync_config_command><!-- PHP function call --></custom_php_resync_config_command>
  <start_command><!-- PHP function call --></start_command>
  <custom_php_service_status_command><!-- PHP function call --></custom_php_service_status_command>
  <custom_php_validation_command><!-- PHP function call --></custom_php_validation_command>
  <custom_php_after_head_command><!-- PHP function call --></custom_php_after_head_command>
  <custom_php_command_before_form><!-- PHP function call --></custom_php_command_before_form>
  <custom_php_after_form_command><!-- PHP function call --></custom_php_after_form_command>
</packagegui>
```

