ansible-role-openvpn-ldap
=============================

Role for deploy openvpn server/client/user with LDAP auth and Certificate Revocation List support.

Attention
-------------

The role released as is, for common understanding how to "OpenVPN with Ansible". This role support most common options, also support
'client config dir' for dynamic work with push routes to client. Role tested only with OpenLDAP.

Ansible versions
--------------------

Role is adapted for Ansible 2.0, tested on 2.1.

Requirements for usage with LDAP
-----------------------------------

**Software:**

* LDAP User Directory;
* SMTP server for send to end-users configuration;
* OpenLDAP (ldapsearch binary) on instance which execute Role, for queries;
* 7zip for compress all end-user files to one archive;


**openvpn-auth-ldap plugin with RFC2307 patch**:

SRPM for EL7 you can found [here](//github.com/k0ste/openvpn-auth-ldap-rfc2307).
PKGBUILD for ArchLinux in [AUR](//aur.archlinux.org/packages/openvpn-auth-ldap).


Example configuration of server with 4 tunnels
---------------------------------------------------

1st instance (openvpn1194) have client-config-dir.
All clients receive routes.
Special clients receive dedicated ipaddr.

```yaml
---
openvpn_client_config_mgmt: 'true'
openvpn_instance: 'server'

openvpn_openvpn1194_config_dir: '/etc/openvpn/openvpn1194'
openvpn_openvpn1194_DEFAULT:
  - 'push route 10.9.0.0 255.255.255.0'
  - 'push route 10.10.0.0 255.255.255.0'
  - 'push route 10.11.0.0 255.255.255.0'

openvpn_dest: '/etc/openvpn/'
openvpn_tls_dest: '/etc/openvpn/tls/'
openvpn_log_dest: '/var/log/openvpn/'
openvpn_pool_dest: '/var/lib/openvpn/'

openvpn_clients_conf:
- {
instance: 'openvpn1194',
dest: "{{ hostvars[inventory_hostname]['openvpn_openvpn1194_config_dir'] }}",
cn: 'DEFAULT',
options: "{{ hostvars[inventory_hostname]['openvpn_openvpn1194_DEFAULT'] }}"
}
- {
instance: 'openvpn1194',
dest: "{{ hostvars[inventory_hostname]['openvpn_openvpn1194_config_dir'] }}",
cn: 'DEFAULT',
options: "{{ hostvars[inventory_hostname]['openvpn_openvpn1194_DEFAULT'] }}"
}
- {
instance: 'openvpn1194',
dest: "{{ hostvars[inventory_hostname]['openvpn_openvpn1194_config_dir'] }}",
cn: 'dallas',
options: 'ifconfig-push 10.15.0.2 255.255.255.0'
}
- {
instance: 'openvpn1194',
dest: "{{ hostvars[inventory_hostname]['openvpn_openvpn1194_config_dir'] }}",
cn: 'boston',
options: 'ifconfig-push 10.15.0.3 255.255.255.0'
}

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
user: "{{ hostvars[inventory_hostname]['openvpn_instance_user'] }}",
group: "{{ hostvars[inventory_hostname]['openvpn_instance_group'] }}",
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
client_config_dir: "{{ hostvars[inventory_hostname]['openvpn_openvpn1194_config_dir'] }}",
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
user: "{{ hostvars[inventory_hostname]['openvpn_instance_user'] }}",
group: "{{ hostvars[inventory_hostname]['openvpn_instance_group'] }}",
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
user: "{{ hostvars[inventory_hostname]['openvpn_instance_user'] }}",
group: "{{ hostvars[inventory_hostname]['openvpn_instance_group'] }}",
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
user: "{{ hostvars[inventory_hostname]['openvpn_instance_user'] }}",
group: "{{ hostvars[inventory_hostname]['openvpn_instance_group'] }}",
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


```yaml
---
openvpn_dest: '/etc/openvpn/'
openvpn_tls_dest: '/etc/openvpn/tls/'
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
user: "{{ hostvars[inventory_hostname]['openvpn_instance_user'] }}",
group: "{{ hostvars[inventory_hostname]['openvpn_instance_group'] }}",
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
user: "{{ hostvars[inventory_hostname]['openvpn_instance_user'] }}",
group: "{{ hostvars[inventory_hostname]['openvpn_instance_group'] }}",
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
-------------------------------------------------------

```yaml
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
user: "{{ hostvars[inventory_hostname]['openvpn_instance_user'] }}",
group: "{{ hostvars[inventory_hostname]['openvpn_instance_group'] }}",
resolv_retry: 'infinite',
explicit_exit_notify: '2',
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

```yaml
---
- name: Deploy OpenVPN Client Instance
  become: true
  gather_facts: true
  remote_user: ansible
  no_log: false
  strategy: free
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
  user: ansible
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
```
