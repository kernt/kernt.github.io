---
tags:
  - konfigurationsmanagement
  - deployment
  - foreman
---
# foreman examples

```
  foreman_url: https://foreman.example.com
  puppetrun: true
  unattended: true
  authentication: true
  passenger: true
  passenger_ruby: /usr/bin/tfm-ruby
  passenger_ruby_package: tfm-rubygem-passenger-native
  plugin_prefix: tfm-rubygem-foreman_
  use_vhost: true
  servername: foreman.example.com
  serveraliases:
  - foreman
  ssl: true
  custom_repo: true
  repo: stable
  configure_epel_repo: true
  configure_scl_repo: true
  selinux: 
  gpgcheck: true
  version: present
  plugin_version: present
  db_manage: true
  db_type: postgresql
  db_adapter: 
  db_host: 
  db_port: 
  db_database: 
  db_username: foreman
  db_password: SYVSSucFfGJb8DqfzmK9faJu3SqD5PYs
  db_sslmode: 
  db_pool: 5
  db_manage_rake: true
  app_root: /usr/share/foreman
  manage_user: true
  user: foreman
  group: foreman
  user_groups:
  - puppet
  rails_env: production
  puppet_home: /var/lib/puppet
  puppet_ssldir: /etc/puppetlabs/puppet/ssl
  locations_enabled: true
  organizations_enabled: true
  passenger_interface: 
  vhost_priority: '05'
  server_port: 80
  server_ssl_port: 443
  server_ssl_ca: /etc/puppetlabs/puppet/ssl/certs/ca.pem
  server_ssl_chain: /etc/puppetlabs/puppet/ssl/certs/ca.pem
  server_ssl_cert: /etc/puppetlabs/puppet/ssl/certs/foreman.example.com.pem
  server_ssl_certs_dir: ''
  server_ssl_key: /etc/puppetlabs/puppet/ssl/private_keys/foreman.example.com.pem
  client_ssl_cert: /etc/puppetlabs/puppet/ssl/certs/foreman.example.com.pem
  client_ssl_key: /etc/puppetlabs/puppet/ssl/private_keys/foreman.example.com.pem
  keepalive: true
  max_keepalive_requests: 100
  keepalive_timeout: 5
  oauth_active: true
  oauth_map_users: false
  oauth_consumer_key: A7n2ENAEaGPSCne7AXBGWuRuXF8EQPfo
  oauth_consumer_secret: x8BmCKvVhFksW25GJKErkJiFpKbRrpoS
  passenger_prestart: true
  passenger_min_instances: 1
  passenger_start_timeout: 600
  admin_username: admin
  admin_password: bQx8CRdCmxxHcqGe
  admin_first_name: Tobias
  admin_last_name: Kern
  admin_email: tobkern1980@googlemail.com
  initial_organization: 
  initial_location: 
  ipa_authentication: true
  http_keytab: /etc/httpd/conf/http.keytab
  pam_service: foreman
  ipa_manage_sssd: true
  websockets_encrypt: true
  websockets_ssl_key: /etc/puppetlabs/puppet/ssl/private_keys/foreman.example.com.pem
  websockets_ssl_cert: /etc/puppetlabs/puppet/ssl/certs/foreman.example.com.pem
  logging_level: info
  loggers: {}
  email_config_method: database
  email_conf: email.yaml
  email_source: email.yaml.erb
  email_delivery_method: 
  email_smtp_address: 
  email_smtp_port: 25
  email_smtp_domain: 
  email_smtp_authentication: none
  email_smtp_user_name: tobkern1980@googlemail.com
  email_smtp_password: Fgh8#sd6$1
foreman_cli:
  foreman_url: 
  version: installed
  manage_root_config: true
  username: 
  password: 
  refresh_cache: false
  request_timeout: 120
  hammer_plugin_prefix: tfm-rubygem-hammer_cli_
foreman_cli_openscap: {}
foreman_proxy:
  repo: stable
  gpgcheck: true
ensure_packages_version: present
  plugin_version: installed
  bind_host: '*'
  http_port: 8000
  ssl_port: 8443
  dir: /usr/share/foreman-proxy
  user: foreman-proxy
  groups: []
  log: /var/log/foreman-proxy/proxy.log
  log_level: INFO
  log_buffer: 2000
  log_buffer_errors: 1000
  http: false
  ssl: true
  ssl_ca: /etc/puppetlabs/puppet/ssl/certs/ca.pem
  ssl_cert: /etc/puppetlabs/puppet/ssl/certs/foreman.example.com.pem
  ssl_key: /etc/puppetlabs/puppet/ssl/private_keys/foreman.example.com.pem
  foreman_ssl_ca: 
  foreman_ssl_cert: 
  foreman_ssl_key: 
  trusted_hosts:
      - foreman.example.com
  ssl_disabled_ciphers: []
  manage_sudoersd: true
  use_sudoersd: true
  puppetca: true
  puppetca_listen_on: https
  ssldir: /etc/puppetlabs/puppet/ssl
  puppetdir: /etc/puppetlabs/puppet
  puppetca_cmd: /opt/puppetlabs/bin/puppet cert
  puppet_group: puppet
  manage_puppet_group: true
  puppet: true
  puppet_listen_on: https
  puppetrun_cmd: /opt/puppetlabs/bin/puppet kick
  puppetrun_provider: 
  customrun_cmd: /bin/false
  customrun_args: -ay -f -s
  mcollective_user: root
  puppetssh_sudo: false
  puppetssh_command: /usr/bin/puppet agent --onetime --no-usecacheonfailure
  puppetssh_user: root
  puppetssh_keyfile: /etc/foreman-proxy/id_rsa
  puppetssh_wait: false
  salt_puppetrun_cmd: puppet.run
  puppet_user: root
  puppet_url: https://foreman.example.com:8140
  puppet_ssl_ca: /etc/puppetlabs/puppet/ssl/certs/ca.pem
  puppet_ssl_cert: /etc/puppetlabs/puppet/ssl/certs/foreman.example.com.pem
  puppet_ssl_key: /etc/puppetlabs/puppet/ssl/private_keys/foreman.example.com.pem
  puppet_use_environment_api: 
  templates: false
  logs: true
  logs_listen_on: https
  tftp: true
  tftp_listen_on: https
  tftp_managed: true
  tftp_manage_wget: true
  tftp_syslinux_filenames:
    - /usr/share/syslinux/chain.c32
    - /usr/share/syslinux/mboot.c32
    - /usr/share/syslinux/menu.c32
    - /usr/share/syslinux/memdisk
    - /usr/share/syslinux/pxelinux.0
  tftp_root: /var/lib/tftpboot
  tftp_dirs:
    - /var/lib/tftpboot/pxelinux.cfg
    - /var/lib/tftpboot/grub
    - /var/lib/tftpboot/grub2
    - /var/lib/tftpboot/boot
    - /var/lib/tftpboot/ztp.cfg
    - /var/lib/tftpboot/poap.cfg
  tftp_servername: 
  dhcp: false
  dhcp_listen_on: https
  dhcp_managed: true
  dhcp_provider: isc
  dhcp_subnets: []
  dhcp_option_domain:
    - example.com
   - home.oshsl.de
  dhcp_search_domains: 
  dhcp_interface: eth0
  dhcp_gateway: 192.168.100.1
  dhcp_range: 
  dhcp_pxeserver: 
  dhcp_nameservers: default
  dhcp_server: 127.0.0.1
  dhcp_config: /etc/dhcp/dhcpd.conf
  dhcp_leases: /var/lib/dhcpd/dhcpd.leases
  dhcp_key_name: 
  dhcp_key_secret: 
  dhcp_omapi_port: 7911
  dns: false
  dns_listen_on: https
  dns_managed: true
  dns_provider: nsupdate
  dns_interface: eth0
  dns_zone: example.com
  dns_reverse: 100.168.192.in-addr.arpa
  dns_server: 127.0.0.1
  dns_ttl: 86400
  dns_tsig_keytab: /etc/foreman-proxy/dns.keytab
  dns_tsig_principal: foremanproxy/foreman.example.com@EXAMPLE.COM
  dns_forwarders: []
  libvirt_network: default
  bmc_listen_on: https
  bmc_default_provider: ipmitool
  realm: false
  realm_listen_on: https
  realm_provider: freeipa
  realm_keytab: /etc/foreman-proxy/freeipa.keytab
  realm_principal: realm-proxy@EXAMPLE.COM
  freeipa_remove_dns: true
  keyfile: /etc/rndc.key
  register_in_foreman: true
  foreman_base_url: https://foreman.example.com
  registered_name: foreman.example.com
  registered_proxy_url: 
  oauth_effective_user: admin
  oauth_consumer_key: A7n2ENAEaGPSCne7AXBGWuRuXF8EQPfo
  oauth_consumer_secret: x8BmCKvVhFksW25GJKErkJiFpKbRrpoS
  puppet_use_cache: 
  puppet:
  version: present
  user: puppet
  group: puppet
  dir: /etc/puppetlabs/puppet
  codedir: /etc/puppetlabs/code
  vardir: /opt/puppetlabs/puppet/cache
  logdir: /var/log/puppetlabs/puppet
  rundir: /var/run/puppetlabs
  ssldir: /etc/puppetlabs/puppet/ssl
  sharedir: /opt/puppetlabs/puppet
  manage_packages: true
  dir_owner: root
  dir_group: 
  package_provider: 
  package_source: 
  port: 8140
  listen: true
  listen_to: []
  pluginsync: true
  splay: false
  splaylimit: '1800'
  autosign: /etc/puppetlabs/puppet/autosign.conf
  autosign_entries: []
  autosign_mode: '0664'
  runinterval: 1800
  usecacheonfailure: true
  runmode: service
  unavailable_runmodes: []
  cron_cmd: 
  systemd_cmd: 
  agent_noop: false
  show_diff: true
  module_repository: 
  configtimeout: 
  ca_server: 
  postrun_command: 
  dns_alt_names: []
  use_srv_records: false
  srv_domain: example.com
  pluginsource: puppet:///plugins
  pluginfactsource: puppet:///pluginfacts
  additional_settings: {}
  agent_additional_settings: {}
  agent_restart_command: /usr/bin/systemctl reload-or-restart puppet
  classfile: $statedir/classes.txt
  hiera_config: $confdir/hiera.yaml
  main_template: puppet/puppet.conf.erb
  agent_template: puppet/agent/puppet.conf.erb
  auth_template: puppet/auth.conf.erb
  allow_any_crl_auth: false
  auth_allowed:
     - $1
  client_package:
    - puppet-agent
  agent: false
  remove_lock: true
  client_certname: foreman.example.com
  puppetmaster: foreman.example.com
  systemd_unit_name: puppet-run
  service_name: puppet
  syslogfacility: 
  environment: production
  server: true
  server_admin_api_whitelist:
    - localhost
    - foreman.example.com
    - srv2.example.com
    - tobkern-desktop.example.com
    - spacewalk.example.com
    - rp4.example.com
  server_user: puppet
  server_group: puppet
  server_dir: /etc/puppetlabs/puppet
  server_ip: 0.0.0.0
  server_port: 8140
  server_ca: true
  server_ca_auth_required: true
  server_ca_client_whitelist:
  - localhost
  - foreman.example.com
    - srv2.example.com
    - tobkern-desktop.example.com
    - spacewalk.example.com
    - rp4.example.com
  server_http: true
  server_http_port: 8139
  server_http_allow: []
  server_reports: foreman
  server_implementation: puppetserver
  server_passenger: true
  server_puppetserver_dir: /etc/puppetlabs/puppetserver
  server_puppetserver_vardir: /opt/puppetlabs/server/data/puppetserver
  server_puppetserver_rundir: /var/run/puppetlabs/puppetserver
  server_puppetserver_logdir: /var/log/puppetlabs/puppetserver
  server_puppetserver_version: 2.6.0
  server_service_fallback: true
```