# ansible-role-openvpn-ldap

Role for deploy OpenVPN server with AD/LDAP and/or OTP plugins support.

## Requirements

* Ansible 3.0.0;
* OpenVPN 2.4+ with builtin `openvpn-client`/`openvpn-server` systemd services;
* SMTP server for send to end-users configuration;
* CA storage, is two options:
 * `file` - this may be tiny Linux VM as delegated host. All certificate issuing
will be maked on this host;
 * `s3` - Amazon S3 bucket. All certificate issuing will be maked on
deployment host;

## Scenarios:

***openvpn_scenario*** variable should be defined, it can be:
* `deploy_instance` - this scenario is for client/server (or both) deployment;
* `deploy_user` - this scenario is for end user, plays like:
  * checks user is present in LDAP;
  * asserts that user `mail` field is e-mail;
  * generates configuration and certs for this user;
    * optionally with inline certs/keys support in one '\*.ovpn' profile for
      compatibility with GUI clients (profile import)
  * makes archive;
  * sends archive to user e-mail address.

## Certificate Revocation List (CRL) support (only when CA storage is S3)

Optionally with this role is possible to revoke generated certs, playbooks
should look like this:

```yaml
---
- name: Deploy Certificate Revocation List
  become: false
  gather_facts: false
  remote_user: ansible
  no_log: false
  strategy: free
  roles:
  - /home/k0ste/sandbox/GIT/ansible-role-openvpn-ldap
  vars_prompt:
   - name: 'openvpn_revoke_target'
     prompt: 'What instance name should be revoked?'
     private: no
  hosts:
  - localhost
```

## Example configuration of server with 4 tunnels

- First instance (openvpn1194) have client-config-dir.
- All clients receive routes.
- Special clients receive dedicated ipaddr.
- Last instance (openvpn1198) allow only one connection per user.

```yaml
---
openvpn_scenario: 'deploy_instance'
openvpn:
- enable: 'true'
  restart: 'true'
  install_package: 'true'
  ca_storage:
# When CA storage is S3:
  - storage_type: 's3'
    storage_source:
# AWS access key id. If not set then the value of the AWS_ACCESS_KEY
# environment variable is used.
    - aws_access_key: "{{ aws_s3_access_key }}"
# AWS secret key. If not set then the value of the AWS_SECRET_KEY environment
# variable is used.
      aws_secret_key: "{{ aws_s3_secret_key }}"
# Bucket name.
      bucket: "{{ aws_s3_bucket }}"
# Asks for server-side encryption or not.
      encrypt: "false"
# Time limit (in seconds) for the URL generated and returned by S3.
      expiration: '120'
# Enable fakeS3.
      rgw: "{{ aws_s3_rgw }}"
# S3 URL endpoint for usage with fakeS3. Otherwise assumes AWS.
      endpoint_url: "{{ endpoint_url }}"
# When set to 'no', SSL certificates will not be validated.
      validate_certs: "{{ omit }}"
# S3 region.
      region: "{{ omit }}"
# Use this boto profile.
      profile: "{{ omit }}"
# Prefix ('folder') where objects placed in bucket, i.e.:
# s3://ansible/ansible-role_openvpn_ldap/
      object_prefix: 'ansible-role_openvpn_ldap'
# Root CA (should be exist).
      root_crt: 'rootCA.crt'
# Key for Root CA (should be exist).
      root_key: 'rootCA.key'
# Prefix where is crl in bucket.
      crl_path: 'crl'
# Prefix where is dhparam/takey in bucket.
      openvpn_path: 'openvpn'
# When CA storage is filesystem:
  ca_storage:
  - storage_type: 'file'
    storage_source:
    - delegate_host: 'mirror.example.ru'
      delegate_user: 'user'
      crl_file: 'crl.pem'
      crl_days: '5000'
      openvpn_path: '/srv/ca/openvpn'
      crl_path: '/srv/ca/crl'
      root_path: '/srv/ca'
      root_crt: 'rootCA.crt'
      root_key: 'rootCA.key'
  systemd_settings:
  - restart_on_failure: 'true'
# Takes one of 'on-success', 'on-failure', 'on-abnormal', 'on-watchdog',
# 'on-abort', or 'always'
    restart_mode: 'always'
# How much burst in interval, seconds.
    start_limit_interval: '30'
# How much times to restart in interval.
    start_limit_burst: '3'
# Configures the time to sleep before restarting a service, seconds
    restart_timeout: '10'
  global_settings:
  - email:
    - host: 'localhost'
      port: '25'
      username: 'user'
      password: 'secret'
      from: 'noc@localhost'
      subject: 'OpenVPN access'
      headers: 'Reply-To=noc@localhost'
      charset: 'utf8'
      body: 'Phoney smile and fake hello'
      timeout: '10'
    csr:
    - country_name: 'US'
      state: 'MA'
      locality_name: 'Boston'
      organization_name: 'Org'
      organization_unit: 'IT'
    ldap:
    # LDAP URI
    - host: 'ldap://ipa.example.com:389'
    # DN to use for simple binding.
      binddn: 'uid=user,cn=users,cn=accounts,dc=example,dc=com'
    # Password to use for simple binding.
      password: 'secret'
    # Base DN for all queries.
      base: 'cn=users,cn=accounts,dc=opentech,dc=local'
    # Scope of queries (one of "base", "onelevel", or "subtree")
      scope: 'onelevel'
    # Filter, default is '(objectClass=*)'.
      filter: 'uid'
      mail_field: 'mail'
    # Use STARTTLS after connecting.
      tls: 'yes'
    # If set to 'no', SSL certificates will not be validated.
      tls_reqcert: 'no'
  openvpn_settings:
  - name: 'openvpn1194'
    type: 'server'
    port: '1194'
    proto: 'udp'
    dev: 'tap0'
    server: '10.15.0.0 255.255.255.0'
    tls_auth: 'ta.key'
    dh: 'dhparam.pem'
# Check peer certificate against the file crl in PEM format. A CRL (certificate
# revocation list) is used when a particular key is compromised but when the
# overall PKI is still intact. Suppose you had a PKI consisting of a CA, root
# certificate, and a number of client certificates. Suppose a laptop computer
# containing a client key and certificate was stolen. By adding the stolen
# certificate to the CRL file, you could reject any connection which attempts
# to use it, while preserving the overall integrity of the PKI. The only time
# when it would be necessary to rebuild the entire PKI from scratch would be if
# the root certificate key itself was compromised.
    crl_verify: 'crl.pem'
    user: 'nobody'
    group: 'nobody'
# Specify whether the client is required to supply a valid certificate.
# Possible options are
#
# 'none' - a client certificate is not required. The client need to
# authenticate using username/password only. Be aware that using this directive
# is less secure than requiring certificates from all clients.
# 'optional' - a client may present a certificate but it is not required to do
# so.
# 'require' - this is the default option. A client is required to present a
# certificate, otherwise VPN access is refused.
    verify_client_cert: 'none'
# Use the authenticated username as the common name, rather than the common name
# from the client cert.
    username_as_common_name: 'true'
    multihome: 'true'
    auth_nocache: 'true'
# Enable a management server on a socket-name Unix socket on those platforms
# supporting it, or on a designated TCP port. For unix sockets, the default
# behaviour is to create a unix domain socket that may be connected to by any
# process. The management interface provides a special mode where the TCP
# management link can operate over the tunnel itself. To enable this mode, set
# IP to tunnel. Tunnel mode will cause the management interface to listen for a
# TCP connection on the local VPN address of the TUN/TAP interface. While the
# management port is designed for programmatic control of OpenVPN by other
# applications, it is possible to telnet to the port, using a telnet client in
# "raw" mode. Once connected, type "help" for a list of commands.
    management: '127.0.0.1 7505'
# Set the TOS field of the tunnel packet to what the payload's TOS is
    pass_tos: 'true'
    persist_key: 'true'
    persist_tun: 'true'
# A helper directive designed to simplify the expression of 'ping' and
# 'ping-restart'. This option can be used on both client and server side, but
# it is enough to add this on the server side as it will push appropriate --ping
# and --ping-restart options to the client. If used on both server and client,
# the values pushed from server will override the client local values. The
# timeout argument will be twice as long on the server side. This ensures that
# a timeout is detected on client side before the server side drops the
# connection. For example, "keepalive: 10 60" expands as follows:
# if mode server:
#   ping 10                    # Argument: interval
#   ping-restart 120           # Argument: timeout*2
#   push "ping 10"             # Argument: interval
#   push "ping-restart 60"     # Argument: timeout
# else
#   ping 10                    # Argument: interval
#   ping-restart 60            # Argument: timeout
    keepalive: '10 60'
# Limit server to a maximum of concurrent clients.
    max_clients: '250'
    reneg_sec: '86400'
    replay_window: '64'
# Because the OpenVPN server mode handles multiple clients through a single tun
# or tap interface, it is effectively a router. The 'client_to_client' flag
# tells OpenVPN to internally route client-to-client traffic rather than
# pushing all client-originating traffic to the TUN/TAP interface. When this
# option is used, each client will 'see' the other clients which are currently
# connected. Otherwise, each client will only see the server. Don't use this
# option if you want to firewall tunnel traffic using custom, per-client rules.
    client_to_client: 'true'
    compress: 'lzo'
    verbosity: '4'
    mute: '10'
# Silence the output of replay warnings, which are a common false alarm on WiFi
# networks. This option preserves the security of the replay protection code
# without the verbosity associated with warnings about duplicate packets.
    mute_replay_warnings: 'true'
# The goal of this option is to provide a long-term association between clients
# (denoted by their common name) and the virtual IP address assigned to them
# from the ifconfig-pool. Maintaining a long-term association is good for
# clients because it allows them to effectively use the –persist-tun option.
# Note that the entries in this file are treated by OpenVPN as suggestions only,
# based on past associations between a common name and IP address. They do not
# guarantee that the given common name will always receive the given IP address.
    ifconfig_pool_persist: 'true'
    status: 'true'
    push:
    - action: 'compress'
      data: 'lzo'
    - action: 'persist-key'
      data: 'true'
    - action: 'persist-tun'
      data: 'true'
    - action: 'dhcp-option'
      data: 'DNS 100.100.101.1'
    client_config_dir: 'true'
    ccd_settings:
    - cn: 'DEFAULT'
      options:
      - action: 'push'
        data: 'route 10.9.0.0 255.255.255.0'
      - action: 'push'
        data: 'route 10.10.0.0 255.255.255.0'
      - action: 'push'
        data: 'route 10.11.0.0 255.255.255.0'
    plugins:
    - otp:
      - enabled: 'true'
        options:
        - debug: 'true'
          password_is_cr: 'true'
          otp_slop: '180'
          totp_t0: '0'
          totp_step: '30'
          totp_digits: '6'
          motp_step: '10'
          hotp_syncwindow: '2'
    - auth_ldap:
      - enabled: 'true'
        options:
        - host: 'ldap://ipa.example.com:389'
          binddn: 'cn=user,cn=users,cn=accounts,dc=example,dc=com'
          password: 'secret'
          timeout: '10'
          follow_referals: 'no'
          tls_enable: 'yes'
          tls_require_cert: 'no'
          base: 'cn=users,cn=accounts,dc=example,dc=com'
          filter: '(uid=%u)'
          require_group: 'true'
          rfc2307: 'false'
          group_base: 'cn=groups,cn=accounts,dc=example,dc=com'
          group_filter: '(cn=vpn_users)'
          group_attr: 'member'
  - name: 'openvpn1195'
    type: 'server'
    port: '1195'
    proto: 'udp'
    dev: 'tap1'
    server: '10.16.0.0 255.255.255.0'
    ca: ''
    tls_auth: 'ta.key'
    dh: 'dhparam.pem'
    crl_verify: 'crl.pem'
    user: 'nobody'
    group: 'nobody'
    verify_client_cert: 'require'
    username_as_common_name: 'false'
    multihome: 'true'
    persist_tun: 'true'
    keepalive: '10 60'
    max_clients: '250'
    reneg_sec: '86400'
    replay_window: '64'
    client_to_client: 'true'
    compress: 'lzo'
    verbosity: '4'
    mute: '10'
    mute_replay_warnings: 'true'
    status: 'true'
    push:
    - action: 'compress'
      data: 'lzo'
    - action: 'persist-key'
      data: 'true'
    - action: 'persist-tun'
      data: 'true'
# Enable TLS and assume server role during TLS handshake. Note that OpenVPN is
# designed as a peer-to-peer application. The designation of client or server
# is only for the purpose of negotiating the TLS control channel.
    tls_server: 'true'
    tls_version_min: '1.2'
    persist_key: 'true'
# Specify a directory dir for custom client config files. After a connecting
# client has been authenticated, OpenVPN will look in this directory for a file
# having the same name as the client’s X509 common name. If a matching file
# exists, it will be opened and parsed for client-specific configuration
# options. If no matching file is found, OpenVPN will instead try to open and
# parse a default file called 'DEFAULT', which may be provided but is not
# required. Note that the configuration files must be readable by the OpenVPN
# process after it has dropped it's root privileges. This file can specify a
# fixed IP address for a given client using 'ifconfig-push', as well as fixed
# subnets owned by the client using 'iroute'. One of the useful properties of
# this option is that it allows client configuration files to be conveniently
# created, edited, or removed while the server is live, without needing to
# restart the server. The following options are legal in a client-specific
# context: 'push', 'push-reset', 'push-remove', 'iroute', 'ifconfig-push',
# config.
    client_config_dir: 'true'
# Require, as a condition of authentication, that a connecting client has a
# 'client_config_dir' file.
    ccd_exclusive: 'true'
    ccd_settings:
    - cn: 'dallas'
      options: 'ifconfig-push 10.15.0.2 255.255.255.0'
  - name: 'openvpn1196'
    type: 'server'
    port: '1196'
    proto: 'udp'
    dev: 'tap2'
    server: '10.17.0.0 255.255.255.0'
    ca: ''
    tls_auth: 'ta.key'
    dh: 'dhparam.pem'
    crl_verify: 'crl.pem'
    user: 'nobody'
    group: 'nobody'
    verify_client_cert: 'require'
    username_as_common_name: 'false'
    multihome: 'true'
    persist_tun: 'true'
    keepalive: '10 60'
    max_clients: '250'
    reneg_sec: '86400'
    replay_window: '64'
    client_to_client: 'true'
    compress: 'lzo'
    verbosity: '4'
    mute: '10'
    mute_replay_warnings: 'true'
    status: 'true'
    push:
    - action: 'compress'
      data: 'lzo'
    - action: 'persist-key'
      data: 'true'
    - action: 'persist-tun'
      data: 'true'
    tls_server: 'true'
    tls_version_min: '1.2'
    persist_key: 'true'
    plugin: 'false'
    client_config_dir: 'true'
    ccd_exclusive: 'true'
    ccd_settings:
    - cn: 'boston'
      options: 'ifconfig-push 10.15.0.3 255.255.255.0'
  - name: 'openvpn1197'
    type: 'server'
    port: '1197'
    proto: 'udp'
    dev: 'tap3'
    server: '10.18.0.0 255.255.255.0'
    ca: ''
    tls_auth: 'ta.key'
    dh: 'dhparam.pem'
    crl_verify: 'crl.pem'
    user: 'nobody'
    group: 'nobody'
    verify_client_cert: 'require'
    username_as_common_name: 'false'
# Allow multiple clients with the same common name to concurrently connect. In
# the absence of this option, OpenVPN will disconnect a client instance upon
# connection of a new client having the same common name.
    duplicate_cn: 'true'
# Configure a multi-homed UDP server. This option needs to be used when a
# server has more than one IP address (e.g. multiple interfaces, or secondary
# IP addresses), and is not using 'local' to force binding to one specific
# address only. This option will add some extra lookups to the packet path to
# ensure that the UDP reply packets are always sent from the address that the
# client is talking to. This is not supported on all platforms, and it adds
# more processing, so it's not enabled by default. Note: this option is only
# relevant for UDP servers.
    multihome: 'true'
    persist_tun: 'true'
    keepalive: '10 60'
    max_clients: '250'
    reneg_sec: '86400'
    replay_window: '64'
    client_to_client: 'true'
    compress: 'lzo'
    verbosity: '4'
    mute: '10'
    mute_replay_warnings: 'true'
    status: 'true'
    push:
    - action: 'compress'
      data: 'lzo'
    - action: 'persist-key'
      data: 'true'
    - action: 'persist-tun'
      data: 'true'
    - action: 'dhcp-option'
      data: 'DNS 100.100.101.1'
    client_config_dir: 'true'
    tls_server: 'true'
    tls_version_min: '1.2'
    persist_key: 'true'
    plugin: 'false'
  - name: 'openvpn1198'
    type: 'server'
    port: '1198'
    proto: 'udp'
    dev: 'tap4'
    server: '10.19.0.0 255.255.255.0'
    ca: ''
    tls_auth: 'ta.key'
    dh: 'dhparam.pem'
    crl_verify: 'crl.pem'
    user: 'nobody'
    group: 'nobody'
    verify_client_cert: 'require'
    username_as_common_name: 'false'
    duplicate_cn: 'false'
    multihome: 'true'
    persist_tun: 'true'
    keepalive: '10 60'
    max_clients: '250'
    reneg_sec: '86400'
    replay_window: '64'
    client_to_client: 'true'
    compress: 'lzo'
    verbosity: '4'
    mute: '10'
    mute_replay_warnings: 'true'
    status: 'true'
    push:
    - action: 'compress'
      data: 'lzo'
    - action: 'persist-key'
      data: 'true'
    - action: 'persist-tun'
      data: 'true'
    - action: 'dhcp-option'
      data: 'DNS 100.100.101.1'
    tls_server: 'true'
    tls_version_min: '1.2'
    persist_key: 'true'
    plugin: 'false'
    client_config_dir: 'true'
    ccd_exclusive: 'true'
    ccd_settings:
    - cn: 'user1'
      options:
      - action: 'ifconfig-push'
        data: '10.12.0.2 255.255.255.0'
      - action: 'push'
        data: 'route 192.168.128.0 255.255.255.0'
```

## Example configuration of client with 2 tunnels

```yaml
---
openvpn:
- openvpn_settings:
  - name: 'tap0'
    type: 'client'
    remote: '10.15.0.1 1195'
    proto: 'udp'
    dev: 'tap0'
    tls_auth: 'ta.key'
# Enable TLS and assume client role during TLS handshake.
    tls_client: 'true'
    tls_version_min: '1.2'
    resolv_retry: 'infinite'
    nobind: 'true'
    ping_exit: '60'
    ping_restart: '60'
    persist_tun: 'true'
    persist_key: 'true'
    remote_random: 'false'
    reneg_sec: '0'
    compress: 'lzo'
    verbosity: '4'
    mute: '5'
    mute_replay_warnings: 'true'
  - name: 'tap1'
    type: 'client'
    remote: '10.16.0.1 1196'
    proto: 'udp'
    dev: 'tap1'
    tls_auth: 'ta.key'
    tls_client: 'true'
    tls_version_min: '1.2'
    resolv_retry: 'infinite'
    nobind: 'true'
    persist_tun: 'true'
    persist_key: 'true'
    remote_random: 'false'
    reneg_sec: '0'
    compress: 'lzo'
    verbosity: '4'
    mute: '5'
    mute_replay_warnings: 'true'
```

## Example configuration of user with auth-login-pass

```yaml
---
openvpn:
- openvpn_settings:
  - name: "tap0"
    type: 'user'
    remote: "openvpnserver.com 1197"
    proto: 'udp'
    dev: 'tap0'
    resolv_retry: 'infinite'
# In UDP client mode or point-to-point mode, send server/peer an exit
# notification if tunnel is restarted or OpenVPN process is exited. In client
# mode, on exit/restart, this option will tell the server to immediately close
# its client instance object rather than waiting for a timeout. The parameter
# (default is '1') controls the maximum number of attempts that the client will
# try to resend the exit notification message. In UDP server mode, send RESTART
# control channel command to connected clients. The parameter (default is '1')
# controls client behavior. With '1' client will attempt to reconnect to the
# same server, with '2' client will advance to the next server. OpenVPN will
# not send any exit notifications unless this option is enabled.
    explicit_exit_notify: '2'
# Do not bind to local address and port. The IP stack will allocate a dynamic
# port for returning packets. Since the value of the dynamic port could not be
# known in advance by a peer, this option is only suitable for peers which will
# be initiating connections by using the 'remote' option.
    nobind: 'true'
    persist_tun: 'true'
# When multiple 'remote' address/ports are specified, or if connection profiles
# are being used, initially randomize the order of the list as a kind of basic
# load-balancing measure.
    remote_random: 'false'
# If specified, this directive will cause OpenVPN to immediately forget
# username/password inputs after they are used. As a result, when OpenVPN needs
# a username/password, it will prompt for input from stdin, which may be
# multiple times during the duration of an OpenVPN session. When using in
# combination with a user/password file and 'chroot' or 'daemon', make sure to
# use an absolute path.
    auth_nocache: 'true'
# Renegotiate data channel key after n seconds (default is '3600)'. When using
# dual-factor authentication, note that this default value may cause the end
# user to be challenged to reauthorize once per hour. Also, keep in mind that
# this option can be used on both the client and server, and whichever uses the
# lower value will be the one to trigger the renegotiation. A common mistake is
# to set 'reneg_sec' to a higher value on either the client or server, while
# the other side of the connection is still using the default value of '3600'
# seconds, meaning that the renegotiation will still occur once per 3600
# seconds. The solution is to increase 'reneg_sec' on both the client and
# server, or set it to 0 on one side of the connection (to disable), and to
# your chosen value on the other side.
    reneg_sec: '0'
# Enable a compression algorithm. The algorithm parameter may be 'lzo', 'lz4'.
# LZO and LZ4 are different compression algorithms, with LZ4 generally offering
# the best performance with least CPU usage. For backwards compatibility with
# OpenVPN versions before v2.4, use 'lzo'.
    compress: 'lzo'
# Set output verbosity (default is '1'). Each level shows all info from the
# previous levels. Level '3' is recommended if you want a good summary of
# what’s happening without being swamped by output. '0' — no output except
# fatal errors.
# '1' to '4' — Normal usage range.
# '5' - output R and W characters to the console for each packet read and
# write, uppercase is used for TCP/UDP packets and lowercase is used for
# TUN/TAP packets.
# '6' to '11' - Debug info range.
    verbosity: '4'
# Log at most n consecutive messages in the same category. This is useful to
# limit repetitive logging of similar message types.
    mute: '5'
    mute_replay_warnings: 'true'
    tls_client: 'true'
    tls_version_min: '1.2'
    tls_auth: 'ta.key'
# Don't re-read key files across SIGUSR1 or 'ping-restart'. This option can be
# combined with "user: 'nobody' to allow restarts triggered by the SIGUSR1
# signal. Normally if you drop root privileges in OpenVPN, the daemon cannot be
# restarted since it will now be unable to re-read protected key files.
# This option solves the problem by persisting keys across SIGUSR1 resets, so
# they don't need to be re-read.
    persist_key: 'true'
# This directive offers policy-level control over OpenVPN's usage of external
# programs and scripts. Lower level values are more restrictive, higher values
# are more permissive
# '0' - strictly no calling of external programs
# '1' - only call built-in executables such as ifconfig, ip, route, or netsh
# (default)
# '2' - allow calling of built-in executables and user-defined scripts
# '3' - allow passwords to be passed to scripts via environmental variables
# (potentially unsafe)
    script_security: '2'
# Run command cmd after successful TUN/TAP device open (pre --user UID change)
# The content of this option will be placed as script in /etc/openvpn dir
    up: ''
# Enable static challenge/response protocol using challenge text.
    static_challenge: 'Enter Google Authenticator Token'
```

## Example playbooks

```yaml
---
- name: Deploy OpenVPN Instance [Cleint\Server]
  become: true
  gather_facts: true
  remote_user: ansible
  no_log: false
  strategy: free
  roles:
  - openvpn_ldap
  hosts:
  - vpn.example.com
```

```yaml
---
- name: Deploy OpenVPN User
  hosts: localhost
  become: false
  gather_facts: false
  remote_user: ansible
  no_log: false
  strategy: linear
  roles:
  - openvpn_ldap
  vars_prompt:
   - name: 'openvpn_target_user'
     prompt: 'What user name?'
     private: no
```
