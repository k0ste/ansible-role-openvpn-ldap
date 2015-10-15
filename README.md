ansible-role-openvpn-ldap
===========================

Role for deploy openvpn server/client/user with LDAP auth and Certificate Revocation List support.

Attention
-------------
The role released as is, for common understanding how to "OpenVPN with Ansible". If someone have questions and/or interest - write to me.

Requirements
---------------

AD/OpenLDAP, SMTP server for send to end-users configuration.
For auth with LDAP need plugin. SRPM for EL7 you can found [here](https://github.com/k0ste/openvpn-auth-ldap-rfc2307).

Example configuration of server with 4 tunnels
-------------------------------------------------

```
---
openvpn_default_routes:
  - '5.128.220.1 255.255.255.255'
  - '5.128.220.254 255.255.255.255'
openvpn_dest: '/etc/openvpn/'
openvpn_tls_dest: '{{ openvpn_dest }}tls/'
openvpn_client_config_dir: '{{ openvpn_dest }}main/'
openvpn_log_dest: '/var/log/openvpn/'
openvpn_pool_dest: '/var/lib/openvpn/'
openvpn_instance: 'server'
openvpn_server:
- {
name: 'openvpn1194',
port: '1194',
proto: 'udp',
dev: 'tap0',
server: '10.15.0.0 255.255.255.0',
ca: '{{ openvpn_ca_file }}',
cert: '{{ openvpn_instance_name }}.crt',
key: '{{ openvpn_instance_name }}.key',
tls_auth: '{{ openvpn_takey_file }}',
dh: '{{ openvpn_dhparam_file }}',
crl_verify: '{{ openvpn_crl_file }}',
user: '{{ openvpn_instance_user }}',
group: '{{ openvpn_instance_group }}',
client_cert_not_required: 'true',
username_as_common_name: 'true',
multihome: 'true',
persist_key: 'true',
persist_tun: 'true',
keepalive: '10 60',
max_clients: '250',
reneg_sec: '86400',
replay_window: '64',
client_to_client: 'true',
comp_lzo: 'adaptive',
verbosity: '4',
mute: '10',
mute_replay_warnings: 'true',
ifconfig_pool_persist: 'pool1194.txt',
status: 'status1194.log',
push: 'true',
push_comp_lzo: 'adaptive',
push_route: '192.168.0.0 255.255.0.0',
push_persist_key: 'true',
push_persist_tun: 'true',
push_dhcp_option: 'DNS 192.168.128.1',
client_config_dir: 'true',
plugin: 'true',
plugin_name: 'openvpn-auth-ldap.so',
plugin_conf: 'auth-ldap.conf'
}
- {
name: 'openvpn1195',
port: '1195',
proto: 'udp',
dev: 'tap1',
server: '10.16.0.0 255.255.255.0',
ca: '{{ openvpn_ca_file }}',
cert: '{{ openvpn_instance_name }}.crt',
key: '{{ openvpn_instance_name }}.key',
tls_auth: '{{ openvpn_takey_file }}',
dh: '{{ openvpn_dhparam_file }}',
crl_verify: '{{ openvpn_crl_file }}',
user: '{{ openvpn_instance_user }}',
group: '{{ openvpn_instance_group }}',
client_cert_not_required: 'false',
username_as_common_name: 'false',
multihome: 'true',
persist_tun: 'true',
keepalive: '10 60',
max_clients: '250',
reneg_sec: '86400',
replay_window: '64',
client_to_client: 'true',
comp_lzo: 'adaptive',
verbosity: '4',
mute: '10',
mute_replay_warnings: 'true',
ifconfig_pool_persist: 'pool1195.txt',
status: 'status1195.log',
push: 'true',
push_comp_lzo: 'adaptive',
push_persist_key: 'true',
push_persist_tun: 'true',
tls_server: 'true',
tls_version_min: '1.2',
persist_key: 'true',
client_config_dir: 'false',
plugin: 'false'
}
- {
name: 'openvpn1196',
port: '1196',
proto: 'udp',
dev: 'tap2',
server: '10.17.0.0 255.255.255.0',
ca: '{{ openvpn_ca_file }}',
cert: '{{ openvpn_instance_name }}.crt',
key: '{{ openvpn_instance_name }}.key',
tls_auth: '{{ openvpn_takey_file }}',
dh: '{{ openvpn_dhparam_file }}',
crl_verify: '{{ openvpn_crl_file }}',
user: '{{ openvpn_instance_user }}',
group: '{{ openvpn_instance_group }}',
client_cert_not_required: 'false',
username_as_common_name: 'false',
multihome: 'true',
persist_tun: 'true',
keepalive: '10 60',
max_clients: '250',
reneg_sec: '86400',
replay_window: '64',
client_to_client: 'true',
comp_lzo: 'adaptive',
verbosity: '4',
mute: '10',
mute_replay_warnings: 'true',
ifconfig_pool_persist: 'pool1196.txt',
status: 'status1196.log',
push: 'true',
push_comp_lzo: 'adaptive',
push_persist_key: 'true',
push_persist_tun: 'true',
tls_server: 'true',
tls_version_min: '1.2',
persist_key: 'true',
client_config_dir: 'false',
plugin: 'false'
}
- {
name: 'openvpn1197',
port: '1197',
proto: 'udp',
dev: 'tap3',
server: '10.18.0.0 255.255.255.0',
ca: '{{ openvpn_ca_file }}',
cert: '{{ openvpn_instance_name }}.crt',
key: '{{ openvpn_instance_name }}.key',
tls_auth: '{{ openvpn_takey_file }}',
dh: '{{ openvpn_dhparam_file }}',
crl_verify: '{{ openvpn_crl_file }}',
user: '{{ openvpn_instance_user }}',
group: '{{ openvpn_instance_group }}',
client_cert_not_required: 'false',
username_as_common_name: 'false',
duplicate_cn: 'true',
multihome: 'true',
persist_tun: 'true',
keepalive: '10 60',
max_clients: '250',
reneg_sec: '86400',
replay_window: '64',
client_to_client: 'true',
comp_lzo: 'adaptive',
verbosity: '4',
mute: '10',
mute_replay_warnings: 'true',
status: 'status1197.log',
push: 'true',
push_comp_lzo: 'adaptive',
push_persist_key: 'true',
push_persist_tun: 'true',
push_dhcp_option: 'DNS 192.168.128.1',
tls_server: 'true',
tls_version_min: '1.2',
persist_key: 'true',
client_config_dir: 'true',
plugin: 'false'
}
```

Example configuration of client with 2 tunnels
-------------------------------------------------


```
---
openvpn_dest: '/etc/openvpn/'
openvpn_tls_dest: '{{ openvpn_dest }}tls/'
openvpn_log_dest: '/var/log/openvpn/'
openvpn_pool_dest: '/var/lib/openvpn/'
openvpn_instance: 'client'
openvpn_client:
- {
name: 'openvpn1195',
remote: '10.15.0.1 1195',
proto: 'udp',
dev: 'tap0',
tls_auth: '{{ openvpn_takey_file }}',
tls_client: 'true',
tls_version_min: '1.2',
ca: '{{ openvpn_ca_file }}',
cert: '{{ openvpn_instance_name }}.crt',
key: '{{ openvpn_instance_name }}.key',
user: '{{ openvpn_instance_user }}',
group: '{{ openvpn_instance_group }}',
resolv_retry: 'infinite',
nobind: 'true',
persist_tun: 'true',
persist_key: 'true',
remote_random: 'false',
reneg_sec: '0',
comp_lzo: 'adaptive',
verbosity: '4',
mute: '5',
mute_replay_warnings: 'true'
}
- {
name: 'openvpn1196',
remote: '10.16.0.1 1196',
proto: 'udp',
dev: 'tap1',
tls_auth: '{{ openvpn_takey_file }}',
tls_client: 'true',
tls_version_min: '1.2',
ca: '{{ openvpn_ca_file }}',
cert: '{{ openvpn_instance_name }}.crt',
key: '{{ openvpn_instance_name }}.key',
user: '{{ openvpn_instance_user }}',
group: '{{ openvpn_instance_group }}',
resolv_retry: 'infinite',
nobind: 'true',
persist_tun: 'true',
persist_key: 'true',
remote_random: 'false',
reneg_sec: '0',
comp_lzo: 'adaptive',
verbosity: '4',
mute: '5',
mute_replay_warnings: 'true'
}

```

Example configuration of user with auth-login-pass
----------------------------------------------------


```
---
openvpn_dest: ''
openvpn_tls_dest: ''
openvpn_log_dest: ''
openvpn_instance: 'user'
openvpn_client:
- {
name: '{{ openvpn_instance_name }}',
remote: 'openvpnserver.com 1197',
proto: 'udp',
dev: 'tap0',
user: '{{ openvpn_instance_user }}',
group: '{{ openvpn_instance_group }}',
resolv_retry: 'infinite',
explicit-exit-notify: '2',
nobind: 'true',
persist_tun: 'true',
remote_random: 'false',
auth_nocache: 'true',
reneg_sec: '0',
comp_lzo: 'adaptive',
verbosity: '4',
mute: '5',
mute_replay_warnings: 'true',
tls_client: 'true',
tls_version_min: '1.2',
tls_auth: '{{ openvpn_takey_file }}',
ca: '{{ openvpn_ca_file }}',
cert: '{{ openvpn_instance_name }}.crt',
key: '{{ openvpn_instance_name }}.key',
persist_key: 'true'
}
```

Example playbooks
----------------------------------------------------

```
---
- name: Deploy OpenVPN Client Instance
  sudo: true
  gather_facts: true
  user: ansible
  roles:
  - { role: packages, packages_list: ['openvpn', 'openssl'] }
  - openvpn_ldap
  hosts: target-client
```

```
---
- name: Deploy Certificate Revocation List
  sudo: true
  gather_facts: true
  user: ansible
  roles:
  - openvpn_ldap
  vars_prompt:
   - name: 'openvpn_revoke_target'
     prompt: 'What instance name should be revoked?'
     private: no
  hosts: target-sever
```

```
---
- name: Deploy OpenVPN Server Instance
  sudo: true
  gather_facts: true
  user: ansible
  roles:
  - { role: packages, packages_list: ['openvpn', 'openssl'] }
  - openvpn_ldap
  hosts: target-server
```

```
---
- name: Deploy OpenVPN User Instance
  hosts: 127.0.0.1
  sudo: true
  gather_facts: true
  user: ansible
  roles:
  - openvpn_ldap
  vars_prompt:
   - name: 'openvpn_target_user'
     prompt: 'What user name?'
     private: no
```
