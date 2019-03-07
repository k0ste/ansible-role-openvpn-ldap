# ansible-role-openvpn-ldap

Role for deploy openvpn server/client/user with LDAP auth and Certificate
Revocation List support.

## Attention

The role released as is, for common understanding how to "OpenVPN with Ansible".
This role support most common options, also support 'client config dir' for
dynamic work with push routes to client. Role tested only with OpenLDAP.

## Requirements for usage with LDAP

* Ansible 2.5+;
* OpenVPN 2.4+ with builtin `openvpn-client`/`openvpn-server` systemd services.
* SMTP server for send to end-users configuration;
* [ansible-plugin-lookup_ldap](//github.com/quinot/ansible-plugin-lookup_ldap)
lookup modules;

## Example configuration of server with 4 tunnels

- First instance (openvpn1194) have client-config-dir.
- All clients receive routes.
- Special clients receive dedicated ipaddr.
- Last instance (openvpn1198) allow only one connection per user.

```yaml
---
openvpn:
- enable: 'true'
  restart: 'true'
  install_package: 'true'
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
    env:
    - dest: '/etc/openvpn'
      tls_dest: '/etc/openvpn/tls'
      log_dest: '/var/log/openvpn'
      pool_dest: '/var/lib/openvpn'
      src: '/srv/ca/openvpn'
      group: 'nobody'
      user: 'nobody'
      dhparam_file: 'dhparam.pem'
      takey_file: 'ta.key'
      client_config_mgmt: 'true'
    ca:
    - delegate_host: 'mirror.example.com'
      delegate_user: 'user'
      crl_path: '/srv/ca/crl'
      crl_file: 'crl.pem'
      crl_days: '5000'
      path: '/srv/ca'
      rootcrt: 'rootCA.crt'
      file: 'rootCA.crt'
      rootkey: 'rootCA.key'
    csr:
    - country_name: 'US'
      state: 'MA'
      locality_name: 'Boston'
      organization_name: 'Org'
      organization_unit: 'IT'
    ldap:
    - host: 'ldap://example.com:389'
      binddn: 'cn=user,ou=people,dc=example,dc=com'
      password: 'secret'
      timeout: '10'
      follow_referals: 'no'
      tls_enable: 'no'
      basedn: 'ou=people,dc=example,dc=com'
      filter: '(uid=%u)'
      require_group: 'true'
      rfc2307: 'false'
      group_basedn: 'ou=groups,dc=example,dc=com'
      group_filter: '(cn=vpn_users)'
      group_attr: 'memberUid'
      user_filter: 'uid'
      mail_filter: 'mail'
  instance_type: 'server'
  server_settings:
  - name: 'openvpn1194'
    port: '1194'
    proto: 'udp'
    dev: 'tap0'
    server: '10.15.0.0 255.255.255.0'
    tls_auth: 'ta.key'
    dh: 'dhparam.pem'
    crl_verify: 'crl.pem'
    user: 'nobody'
    group: 'nobody'
    verify_client_cert: 'none'
    username_as_common_name: 'true'
    multihome: 'true'
    management: '127.0.0.1 7505'
    persist_key: 'true'
    persist_tun: 'true'
    keepalive: '10 60'
    max_clients: '250'
    reneg_sec: '86400'
    replay_window: '64'
    client_to_client: 'true'
    comp_lzo: 'adaptive'
    verbosity: '4'
    mute: '10'
    mute_replay_warnings: 'true'
    ifconfig_pool_persist: 'pool1194.txt'
    status: 'status1194.log'
    push: 'true'
    push_comp_lzo: 'adaptive'
    push_persist_key: 'true'
    push_persist_tun: 'true'
    push_dhcp_option: 'DNS 100.100.101.1'
    client_config_dir: 'true'
    plugin: 'true'
    plugin_name: 'openvpn-auth-ldap.so'
    plugin_conf: 'auth-ldap.conf'
  - name: 'openvpn1195'
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
    comp_lzo: 'adaptive'
    verbosity: '4'
    mute: '10'
    mute_replay_warnings: 'true'
    status: 'status1195.log'
    push: 'true'
    push_comp_lzo: 'adaptive'
    push_persist_key: 'true'
    push_persist_tun: 'true'
    client_config_dir: 'true'
    ccd_exclusive: 'true'
    tls_server: 'true'
    tls_version_min: '1.2'
    persist_key: 'true'
    plugin: 'false'
  - name: 'openvpn1196'
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
    comp_lzo: 'adaptive'
    verbosity: '4'
    mute: '10'
    mute_replay_warnings: 'true'
    status: 'status1196.log'
    push: 'true'
    push_comp_lzo: 'adaptive'
    push_persist_key: 'true'
    push_persist_tun: 'true'
    client_config_dir: 'true'
    ccd_exclusive: 'true'
    tls_server: 'true'
    tls_version_min: '1.2'
    persist_key: 'true'
    plugin: 'false'
  - name: 'openvpn1197'
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
    duplicate_cn: 'true'
    multihome: 'true'
    persist_tun: 'true'
    keepalive: '10 60'
    max_clients: '250'
    reneg_sec: '86400'
    replay_window: '64'
    client_to_client: 'true'
    comp_lzo: 'adaptive'
    verbosity: '4'
    mute: '10'
    mute_replay_warnings: 'true'
    status: 'status1197.log'
    push: 'true'
    push_comp_lzo: 'adaptive'
    push_persist_key: 'true'
    push_persist_tun: 'true'
    push_dhcp_option: 'DNS 100.100.101.1'
    client_config_dir: 'true'
    tls_server: 'true'
    tls_version_min: '1.2'
    persist_key: 'true'
    plugin: 'false'
  - name: 'openvpn1198'
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
    comp_lzo: 'adaptive'
    verbosity: '4'
    mute: '10'
    mute_replay_warnings: 'true'
    status: 'status1198.log'
    push: 'true'
    push_comp_lzo: 'adaptive'
    push_persist_key: 'true'
    push_persist_tun: 'true'
    push_dhcp_option: 'DNS 100.100.101.1'
    client_config_dir: 'true'
    ccd_exclusive: 'true'
    tls_server: 'true'
    tls_version_min: '1.2'
    persist_key: 'true'
    plugin: 'false'

openvpn_openvpn1194_DEFAULT:
- 'push "route 10.9.0.0 255.255.255.0"'
- 'push "route 10.10.0.0 255.255.255.0"'
- 'push "route 10.11.0.0 255.255.255.0"'

openvpn_openvpn1198_user1:
- 'ifconfig-push "10.19.0.2 255.255.255.0"'
- 'push "route 192.168.128.0 255.255.255.0"'

openvpn_clients_conf:
- instance: 'openvpn1194'
  cn: 'DEFAULT'
  options: "{{ hostvars[inventory_hostname]['openvpn_openvpn1194_DEFAULT'] }}"
- instance: 'openvpn1195'
  cn: 'dallas'
  options: 'ifconfig-push 10.15.0.2 255.255.255.0'
- instance: 'openvpn1196'
  cn: 'boston'
  options: 'ifconfig-push 10.15.0.3 255.255.255.0'
## Restricted users. Have only one connection
- instance: 'openvpn1198'
  cn: 'user1'
  options: "{{ hostvars[inventory_hostname]['openvpn_openvpn1198_user1'] }}"
```

## Example configuration of client with 2 tunnels

```yaml
---
openvpn:
- instance_type: 'client'
  client_settings:
  - name: 'tap0'
    remote: '10.15.0.1 1195'
    proto: 'udp'
    dev: 'tap0'
    tls_auth: 'ta.key'
    tls_client: 'true'
    tls_version_min: '1.2'
    resolv_retry: 'infinite'
    nobind: 'true'
    persist_tun: 'true'
    persist_key: 'true'
    remote_random: 'false'
    reneg_sec: '0'
    comp_lzo: 'adaptive'
    verbosity: '4'
    mute: '5'
    mute_replay_warnings: 'true'
  - name: 'tap1'
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
    comp_lzo: 'adaptive'
    verbosity: '4'
    mute: '5'
    mute_replay_warnings: 'true'
```

## Example configuration of user with auth-login-pass

```yaml
---
openvpn:
- instance_type: 'user'
  client_settings:
  - name: "{{ vars['openvpn_instance_name'] }}"
    remote: "openvpnserver.com 1197"
    proto: 'udp'
    dev: 'tap0'
    resolv_retry: 'infinite'
    explicit_exit_notify: '2'
    nobind: 'true'
    persist_tun: 'true'
    remote_random: 'false'
    auth_nocache: 'true'
    reneg_sec: '0'
    comp_lzo: 'adaptive'
    verbosity: '4'
    mute: '5'
    mute_replay_warnings: 'true'
    tls_client: 'true'
    tls_version_min: '1.2'
    tls_auth: 'ta.key'
    persist_key: 'true'
```

## Example playbooks

```yaml
---
- name: Deploy OpenVPN Client Instance
  become: true
  gather_facts: true
  remote_user: ansible
  no_log: false
  strategy: free
  vars:
    openvpn_target_port: ''
  roles:
  - openvpn_ldap
  hosts: target-client
```

```yaml
---
- name: Deploy Certificate Revocation List
  become: true
  gather_facts: true
  remote_user: ansible
  no_log: false
  strategy: free
  roles:
  - openvpn_ldap
  vars:
    openvpn_target_port: ''
  vars_prompt:
   - name: 'openvpn_revoke_target'
     prompt: 'What instance name should be revoked?'
     private: no
  hosts: target-sever
```

```yaml
---
- name: Deploy OpenVPN Server Instance
  become: true
  gather_facts: true
  remote_user: ansible
  no_log: false
  strategy: free
  vars:
    openvpn_target_port: ''
  roles:
  - openvpn_ldap
  hosts: target-server
```

```yaml
---
- name: Deploy OpenVPN User Instance
  hosts: 127.0.0.1
  become: true
  gather_facts: true
  remote_user: ansible
  no_log: false
  strategy: free
  roles:
  - openvpn_ldap
  vars_prompt:
   - name: 'openvpn_target_user'
     prompt: 'What user name?'
     private: no
   - name: 'openvpn_target_port'
     prompt: 'What port (1197 for users, 1198 for restricted_users)?'
     private: no
     default: '1197'
```
