# ansible-role-openvpn-ldap

Role for deploy OpenVPN server with AD/LDAP and/or OTP plugins support.

## Requirements for usage with LDAP

* Ansible 2.7+
* OpenVPN 2.4+ with builtin `openvpn-client`/`openvpn-server` systemd services.
* SMTP server for send to end-users configuration;
* [ansible-plugin-lookup_ldap](//github.com/quinot/ansible-plugin-lookup_ldap)
lookup modules.
* CA storage, is two options:
 * `file` - this may be tiny Linux VM as delegated host. All certificate issuing
will be maked on this host.
 * `s3` - Amazon S3 bucket. All certificate issuing will be maked on
deployment host.

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
      s3_url: "{{ aws_s3_url }}"
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
      log_dest: '/var/log/openvpn'
      group: 'nobody'
      user: 'nobody'
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
      basedn: 'ou=people,dc=example,dc=com'
      filter: 'uid'
      mail_field: 'mail'
  openvpn_settings:
  - name: 'openvpn1194'
    type: 'server'
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
    auth_nocache: 'true'
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
    ifconfig_pool_persist: 'true'
    status: 'true'
    push: 'true'
    push_comp_lzo: 'adaptive'
    push_persist_key: 'true'
    push_persist_tun: 'true'
    push_dhcp_option: 'DNS 100.100.101.1'
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
    comp_lzo: 'adaptive'
    verbosity: '4'
    mute: '10'
    mute_replay_warnings: 'true'
    status: 'true'
    push: 'true'
    push_comp_lzo: 'adaptive'
    push_persist_key: 'true'
    push_persist_tun: 'true'
    tls_server: 'true'
    tls_version_min: '1.2'
    persist_key: 'true'
    client_config_dir: 'true'
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
    comp_lzo: 'adaptive'
    verbosity: '4'
    mute: '10'
    mute_replay_warnings: 'true'
    status: 'true'
    push: 'true'
    push_comp_lzo: 'adaptive'
    push_persist_key: 'true'
    push_persist_tun: 'true'
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
    status: 'true'
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
    comp_lzo: 'adaptive'
    verbosity: '4'
    mute: '10'
    mute_replay_warnings: 'true'
    status: 'true'
    push: 'true'
    push_comp_lzo: 'adaptive'
    push_persist_key: 'true'
    push_persist_tun: 'true'
    push_dhcp_option: 'DNS 100.100.101.1'
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
    comp_lzo: 'adaptive'
    verbosity: '4'
    mute: '5'
    mute_replay_warnings: 'true'
```

## Example configuration of user with auth-login-pass

```yaml
---
openvpn:
- openvpn_settings:
  - name: "{{ vars['openvpn_instance_name'] }}"
    type: 'user'
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
