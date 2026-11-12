# Ansible Role: valkey

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-valkey)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-valkey)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-valkey)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-valkey/dev.yml?branch=dev&label=dev)](https://github.com/jomrr/ansible-role-valkey/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-valkey/main.yml?branch=main&label=main)](https://github.com/jomrr/ansible-role-valkey/actions/workflows/main.yml?query=branch%3Amain)

Install and configure Valkey as a secure local or network cache.

## Purpose

Install, configure, enable and start a standalone Valkey cache with independent
TCP and Unix socket access.

## Scope

### Managed

- Distribution packages, service configuration, cache data directory and systemd
  overrides.
- Independent Unix socket and TCP listeners, password and named ACL users,
  optional TLS using host certificate files.
- Memory limits, eviction policy, optional RDB/AOF persistence and additional
  native directives.
- Socket group membership, SELinux port labels and a narrowly scoped policy for
  configured local client domains.

### Not Managed

- Firewall rules, certificate creation, replication, Sentinel and clusters.
- Client services, certificate renewal and removal of previous socket group
  memberships or SELinux port labels.

## Requirements

- TLS requires PEM certificate, private key and CA files readable by valkey and
  permitted by host MAC policy; the role neither creates nor copies them.
- SELinux hosts require the distribution policy defining redis_t, redis_port_t
  and redis_var_run_t; configured client domains must exist.

## Dependencies

```yaml
collections:
  - name: community.general
    version: '>=12.0.0'
  - name: ansible.posix
    version: '>=2.0.0'
```

## Role Variables

### `valkey_tcp_enabled`

Type: `bool`. Required: `false`.

Enable TCP independently of the Unix socket.

Default:

```yaml
valkey_tcp_enabled: true
```

### `valkey_bind_addresses`

Type: `list`. Required: `false`.

Numeric TCP listen addresses, optionally prefixed with minus; non-loopback
listeners require authentication.

Default:

```yaml
valkey_bind_addresses:
  - 127.0.0.1
  - -::1
```

### `valkey_tcp_port`

Type: `int`. Required: `false`.

TCP port from 1 to 65535; TLS replaces plaintext on this port when enabled.

Default:

```yaml
valkey_tcp_port: 6379
```

### `valkey_socket_enabled`

Type: `bool`. Required: `false`.

Enable the Unix socket independently of TCP.

Default:

```yaml
valkey_socket_enabled: false
```

### `valkey_socket_path`

Type: `str`. Required: `false`.

Absolute socket path in a dedicated directory; the role owns its parent and
grants traversal to valkey-clients.

Default:

```yaml
valkey_socket_path: /run/valkey/valkey.sock
```

### `valkey_socket_mode`

Type: `str`. Required: `false`.

Quoted octal socket permissions with owner access and no permissions for other
users.

Default:

```yaml
valkey_socket_mode: '0770'
```

### `valkey_socket_users`

Type: `list`. Required: `false`.

Local service users added to valkey-clients; restart their services to activate
supplementary group changes.

Default:

```yaml
valkey_socket_users: []
```

### `valkey_tls_enabled`

Type: `bool`. Required: `false`.

Enable TLS on the TCP listener, disabling plaintext TCP.

Default:

```yaml
valkey_tls_enabled: false
```

### `valkey_tls_cert_file`

Type: `str`. Required: `false`.

Absolute path to the PEM server certificate, readable by the valkey service
user.

Default:

```yaml
valkey_tls_cert_file: ''
```

### `valkey_tls_key_file`

Type: `str`. Required: `false`.

Absolute path to the PEM private key, readable by the valkey service user.

Default:

```yaml
valkey_tls_key_file: ''
```

### `valkey_tls_ca_file`

Type: `str`. Required: `false`.

Absolute path to the PEM CA bundle used for TLS peer verification.

Default:

```yaml
valkey_tls_ca_file: ''
```

### `valkey_tls_auth_clients`

Type: `bool`. Required: `false`.

Require a trusted client certificate in addition to password or ACL
authentication.

Default:

```yaml
valkey_tls_auth_clients: true
```

### `valkey_password`

Type: `str`. Required: `false`.

Password for the default user; stored as a SHA-256 ACL hash. Empty disables the
default user when named ACL users exist.

Default:

```yaml
valkey_password: ''
```

### `valkey_acl_users`

Type: `list`. Required: `false`.

Each named user requires name and a nonempty password; the name default is
reserved.
Optional commands, key_patterns and channels override valkey_acl_commands,
valkey_acl_key_patterns and valkey_acl_channels.
Command permissions start with plus or minus, key patterns with a tilde and
Pub/Sub channel patterns with an ampersand.

Default:

```yaml
valkey_acl_users: []
```

### `valkey_maxmemory`

Type: `str`. Required: `false`.

Memory limit in bytes or with k, kb, m, mb, g or gb suffix; zero removes the
limit.

Default:

```yaml
valkey_maxmemory: 256mb
```

### `valkey_maxmemory_policy`

Type: `str`. Required: `false`.

Eviction policy applied when the memory limit is reached.

Default:

```yaml
valkey_maxmemory_policy: allkeys-lru
```

### `valkey_rdb_enabled`

Type: `bool`. Required: `false`.

Enable scheduled RDB snapshots; disabled by default.

Default:

```yaml
valkey_rdb_enabled: false
```

### `valkey_rdb_save`

Type: `list`. Required: `false`.

Snapshot intervals used when RDB is enabled; each entry requires positive
seconds and changed-key count.

Default:

```yaml
valkey_rdb_save:
  - seconds: 900
    changes: 1
  - seconds: 300
    changes: 10
  - seconds: 60
    changes: 10000
```

### `valkey_aof_enabled`

Type: `bool`. Required: `false`.

Enable append-only persistence; disabled by default.

Default:

```yaml
valkey_aof_enabled: false
```

### `valkey_aof_fsync`

Type: `str`. Required: `false`.

Append-only file synchronization policy.

Default:

```yaml
valkey_aof_fsync: everysec
```

### `valkey_extra_directives`

Type: `list`. Required: `false`.

Additional native directives with individually quoted arguments; managed
directives, includes and modules are forbidden.

Default:

```yaml
valkey_extra_directives: []
```

### `valkey_selinux_socket_clients`

Type: `list`. Required: `false`.

SELinux domains permitted to connect to the socket, such as httpd_t; requires
the platform redis SELinux policy.

Default:

```yaml
valkey_selinux_socket_clients: []
```

### `valkey_acl_commands`

Type: `list`. Required: `false`.

Default commands permissions for ACL users without an item override.

Default:

```yaml
valkey_acl_commands:
  - +@read
  - +@write
  - +ping
```

### `valkey_acl_key_patterns`

Type: `list`. Required: `false`.

Default keys permissions for ACL users without an item override.

Default:

```yaml
valkey_acl_key_patterns:
  - ~*
```

### `valkey_acl_channels`

Type: `list`. Required: `false`.

Default channels permissions for ACL users without an item override.

Default:

```yaml
valkey_acl_channels: []
```

## Managed Files

- `/etc/valkey/valkey.conf`
- `/etc/systemd/system/<service>.d/override.conf`
- `/etc/tmpfiles.d/valkey.conf`
- `/etc/selinux/valkey_socket_clients.cil`
- `/var/lib/valkey`

## Check Mode

Check mode predicts changes on a provisioned host without changing configuration
or restarting the service.

- A first installation in check mode cannot validate resources dependent on
  packages and service accounts that do not yet exist.

## Service Behavior

Configuration changes restart Valkey; unchanged runs leave the service running.
A stopped or disabled service is restored.

### Handlers

- Reload systemd before restarting after unit changes.
- Restart the server after configuration changes; cache contents are lost unless
  persistence is enabled.

## Security Notes

- Defaults bind to 127.0.0.1 and optional ::1 with protected mode enabled. TCP
  on any other address requires a password or named ACL users.
- With named ACL users and no default password, the default user is disabled. A
  password and named users can be combined.
- TLS replaces plaintext on the configured TCP port. Client certificates are
  required by default and do not replace password/ACL authentication.
- Socket clients join valkey-clients, not the private valkey group; they cannot
  read the configuration or cache data. Restart client services after group
  changes.
- Authentication and configuration tasks suppress secret output and diffs. Use
  Ansible Vault or another secret source for credentials.
- The configuration is root-owned with mode 0640 and group valkey. The data
  directory is private to valkey with mode 0700.
- Additional directives cannot override managed listeners, TLS, ACLs,
  persistence or filesystem paths, include files, load modules or rename
  commands.

## Operational Notes

- Supported distributions are AlmaLinux, Debian, Fedora, openSUSE Leap, openSUSE
  Tumbleweed and Ubuntu. Valkey packages must be available for the chosen
  release.
- Debian and Ubuntu use `valkey-server.service`, RedHat-family distributions use
  `valkey.service`, and openSUSE uses `valkey@valkey.service`.
- At least one transport must remain enabled. Socket paths must use a dedicated
  parent directory and fit the Unix socket path limit.
- Socket ancestors must permit client traversal. The role owns the socket parent
  with mode 0750 and group valkey-clients.
- RDB and AOF are off by default. Disabling persistence does not delete existing
  RDB/AOF files; remove obsolete data explicitly when a fresh cache is required.
- Valkey has no offline configuration validator. Native syntax and TLS material
  are checked when the service starts; invalid extra directives cause startup
  failure.
- The SELinux policy grants directory traversal, socket-file access and stream
  connection only to the configured domains. An empty domain list revokes these
  custom grants.
- Changing ports or socket paths leaves previous SELinux mappings and
  directories intact. Manage cleanup explicitly after clients have migrated.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
| --------- | ------------ | ------- | --------------- |
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest](https://hub.docker.com/r/jomrr/molecule-almalinux) |
| Debian | Debian | latest | [jomrr/molecule-debian:latest](https://hub.docker.com/r/jomrr/molecule-debian) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest](https://hub.docker.com/r/jomrr/molecule-fedora) |
| Suse | OpenSuse Leap | latest | [jomrr/molecule-opensuse-leap:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-leap) |
| Suse | OpenSuse Tumbleweed | latest | [jomrr/molecule-opensuse-tumbleweed:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-tumbleweed) |
| Debian | Ubuntu | latest | [jomrr/molecule-ubuntu:latest](https://hub.docker.com/r/jomrr/molecule-ubuntu) |

## Example Playbook

### Loopback cache

Default 256 MB cache, allkeys-lru eviction and no persistence.

```yaml
---
- name: Configure a local cache
  hosts: cache
  gather_facts: true
  roles:
    - role: jomrr.valkey
```

### Unix socket for a confined web service

Socket-only cache; restart the local web service after group changes.

```yaml
---
- name: Configure a web application cache
  hosts: web
  gather_facts: true
  roles:
    - role: jomrr.valkey
      valkey_tcp_enabled: false
      valkey_socket_enabled: true
      valkey_socket_path: /run/valkey/valkey.sock
      valkey_socket_users: [apache]
      valkey_selinux_socket_clients: [httpd_t]
```

### TLS and named ACL users

TLS on port 6380 with client certificates and an application ACL.

```yaml
---
- name: Configure an authenticated cache
  hosts: cache
  gather_facts: true
  roles:
    - role: jomrr.valkey
      valkey_bind_addresses: [0.0.0.0]
      valkey_tcp_port: 6380
      valkey_tls_enabled: true
      valkey_tls_cert_file: /etc/valkey/tls/server.crt
      valkey_tls_key_file: /etc/valkey/tls/server.key
      valkey_tls_ca_file: /etc/valkey/tls/ca.crt
      valkey_acl_users:
        - name: application
          password: "{{ vault_valkey_application_password }}"
          commands: ['+@read', '+@write', '+ping']
          key_patterns: ['~application:*']
      valkey_extra_directives:
        - name: timeout
          arguments: ['300']
```

## References

- [Valkey configuration](https://valkey.io/topics/valkey.conf/)
- [Valkey ACLs](https://valkey.io/topics/acl/)
- [Valkey TLS](https://valkey.io/topics/tls/)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2026 Jonas Mauer.
