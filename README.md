# Ansible Role: mosquitto

[![CI](https://github.com/xxthunder/ansible-role-mosquitto/actions/workflows/test.yml/badge.svg)](https://github.com/xxthunder/ansible-role-mosquitto/actions/workflows/test.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Ansible](https://img.shields.io/badge/Ansible-2.11%2B-blue.svg)](https://docs.ansible.com/)
[![ansible-lint](https://img.shields.io/badge/ansible--lint-passing-brightgreen.svg)](https://ansible.readthedocs.io/projects/lint/)

Install and configure a [Mosquitto](https://mosquitto.org/) MQTT broker on a
Debian host. The role:

- Installs the `mosquitto` apt package and manages its systemd service.
- Configures a listener (all interfaces by default, so it's LAN-reachable).
- Requires authentication — a **password file** with `allow_anonymous false` by
  default; accounts are supplied by the caller (keep them in your vault).
- Leaves persistence and logging at the distro defaults from the base
  `mosquitto.conf` (on Debian: persistence enabled at `/var/lib/mosquitto`).

## Requirements

- A Debian host (bookworm/trixie) with `systemd` and `mosquitto` installable via
  apt.
- At least one entry in `mosquitto_users` (otherwise, with anonymous disabled, no
  client can connect).

## Role Variables

| Variable | Default | Description |
|---|---|---|
| `mosquitto_users` | `[]` | List of `{ username, password }`. Supply via vault. Required in practice (see above). |
| `mosquitto_package` | `mosquitto` | Debian package name. |
| `mosquitto_version` | `""` | Optional pinned apt version; empty tracks current. |
| `mosquitto_listen_port` | `1883` | Listener TCP port. |
| `mosquitto_listen_address` | `""` | Bind address; empty = all interfaces. |
| `mosquitto_allow_anonymous` | `false` | Allow unauthenticated clients. |
| `mosquitto_password_file` | `/etc/mosquitto/passwd` | Password file path. |
| `mosquitto_conf_file` | `/etc/mosquitto/conf.d/ansible.conf` | Role-managed listener config. |
| `mosquitto_user` / `mosquitto_group` | `mosquitto` | Owner of the password file. |
| `mosquitto_service` | `mosquitto` | systemd service name. |

`meta/argument_specs.yml` is the authoritative variable surface and is validated
at role start.

> **Password rotation:** users are added idempotently, but a password *change*
> for an existing user is **not** re-applied (re-running `mosquitto_passwd` would
> never converge, since it re-salts each call). To rotate, remove the user's line
> from the password file (or delete the file) and re-run.

## Example Playbook

```yaml
- hosts: broker
  become: true
  vars:
    mosquitto_users:
      - username: zigbee2mqtt
        password: "{{ vault_mqtt_zigbee2mqtt_password }}"
      - username: openhab
        password: "{{ vault_mqtt_openhab_password }}"
  roles:
    - mosquitto
```

## Testing

The `default` molecule scenario converges the role in a systemd-enabled Debian
container and idempotency-checks it, then verifies the service is active and the
listener is up.

## License

MIT
