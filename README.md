# ansible-role-fail2ban

[![CI](https://github.com/pluggero/ansible-role-fail2ban/actions/workflows/ci.yml/badge.svg)](https://github.com/pluggero/ansible-role-fail2ban/actions/workflows/ci.yml) [![Ansible Galaxy downloads](https://img.shields.io/ansible/role/d/pluggero/fail2ban?label=Galaxy%20downloads&logo=ansible&color=%23096598)](https://galaxy.ansible.com/ui/standalone/roles/pluggero/fail2ban)

Installs and configures fail2ban: `fail2ban.local`, `jail.local` ([DEFAULT]), per-jail `jail.d/<name>.conf` files, and the `fail2ban` service.

## Requirements

**Collections:**

- ansible.builtin (included with Ansible)
- community.general (pacman module, Archlinux only)

**Role dependencies:**

- none

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
fail2ban_install_method: "package"
fail2ban_config_dir: "/etc/fail2ban"
fail2ban_service_enabled: true
fail2ban_service_state: "started"
```

Installation, paths and service management. `fail2ban_install_method` only supports `"package"` in this role.

```yaml
fail2ban_loglevel: "INFO"
fail2ban_logtarget: "/var/log/fail2ban.log"
```

Rendered into `fail2ban.local` `[Definition]`.

```yaml
fail2ban_bantime: 600 # seconds
fail2ban_findtime: 600 # seconds
fail2ban_maxretry: 5
fail2ban_backend: "auto"
fail2ban_destemail: "root@localhost"
fail2ban_sender: "root@{{ ansible_facts['fqdn'] }}"
fail2ban_action: "%(action_)s"
fail2ban_banaction: ""
fail2ban_ignoreself: "true"
fail2ban_ignoreips:
  - "127.0.0.1/8 ::1"
```

Rendered into `jail.local` `[DEFAULT]`.

```yaml
fail2ban_configuration: []
# - option: socket
#   value: /var/run/fail2ban/fail2ban.sock
#   section: Definition

fail2ban_jail_configuration: []
# - option: ignoreself
#   value: "true"
#   section: DEFAULT
```

Escape hatches for keys that don't have a dedicated variable. `fail2ban_configuration` appends to `fail2ban.local`; `fail2ban_jail_configuration` appends to `jail.local` `[DEFAULT]` only.

```yaml
# fail2ban_filterd_path: "files/filter.d"
```

Controller-side directory of custom filter definitions to copy into `filter.d/`. Unset (default) to skip.

```yaml
fail2ban_jails:
  - name: sshd
    enabled: true
    filter: sshd
    port: ssh
    logpath: /var/log/auth.log
    maxretry: 5
    findtime: 600
    bantime: 3600
```

Each entry renders to its own `jail.d/<name>.conf`. Schema:

| Field | Type | Required | Notes |
|---|---|---|---|
| `name` | string | yes | `^[A-Za-z0-9_.-]+$`, unique; becomes `jail.d/<name>.conf` and `[<name>]` |
| `enabled` | bool | no (default `true`) | |
| `filter` | string | no (defaults to `name`) | |
| `port` | string/int | no | |
| `logpath` | string or list | no | omit when using the systemd backend |
| `backend` | string | no | inherits `[DEFAULT]` if omitted |
| `maxretry`, `findtime`, `bantime`, `action` | — | no | inherit `[DEFAULT]` if omitted |
| `ignoreip` | list of strings | no | inherits `[DEFAULT]` if omitted |
| `extra_options` | dict | no | verbatim `key = value` lines |

**Known limitation:** removing an entry from `fail2ban_jails` does not delete its `jail.d/<name>.conf` file. Use `enabled: false` to disable a jail instead.

## Example Playbook

```yaml
- hosts: all
  roles:
    - role: pluggero.fail2ban
      vars:
        fail2ban_destemail: "security@example.com"
        fail2ban_jails:
          - name: sshd
            port: ssh
            logpath: /var/log/auth.log
            maxretry: 5
          - name: nginx-http-auth
            port: "http,https"
            logpath: /var/log/nginx/error.log
            maxretry: 6
            extra_options:
              mode: aggressive
```

## License

MIT

## Author Information

This role was created in 2026 by pluggero.
