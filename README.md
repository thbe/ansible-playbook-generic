# ansible-playbook-generic

[![Linter](https://github.com/thbe/ansible-playbook-generic/actions/workflows/linter.yml/badge.svg)](https://github.com/thbe/ansible-playbook-generic/actions/workflows/linter.yml)

Collection of small, generic day-to-day operations playbooks for managing a
fleet of Linux nodes. These playbooks are intentionally minimal and are meant
to be run ad hoc against an inventory you provide.

This repository is normally consumed as the `playbooks/generic` git submodule of
[ansible-main](https://github.com/thbe/ansible-main), but it can also be used
standalone.

## Table of Contents

- [Requirements](#requirements)
- [Playbooks](#playbooks)
- [Usage](#usage)
- [Notes](#notes)
- [License](#license)
- [Author](#author)

## Requirements

- Ansible 2.14+ (core) with the following collections:
  - `ansible.posix`
  - `community.general`
- An inventory file (not shipped with this repo; `inventories/` is git-ignored).
- SSH access to the target hosts with privilege escalation (`become`) available.

All playbooks target the `all` host group. Restrict the scope with `--limit`.

## Playbooks

| Playbook                    | Privilege | Facts | Description                                                                                             |
| --------------------------- | --------- | ----- | ------------------------------------------------------------------------------------------------------- |
| `download_hosts_files.yml`  | per-task  | yes   | Reads `/etc/hosts` from each host and stores a copy locally under `tmp/<hostname>_hosts`.                |
| `environment_dump.yml`      | yes       | yes   | Dumps `vars`, `environment`, `groups` and `hostvars` for debugging (tag: `debug_info`).                 |
| `ntpdate.yml`               | yes       | no    | Forces an immediate time sync via `chronyd -q`, then restarts and enables `chronyd`.                    |
| `reboot.yml`                | yes       | no    | Schedules a reboot one minute in the future with an operator warning message.                            |
| `shutdown.yml`              | yes       | no    | Schedules a halt one minute in the future with an operator warning message.                              |
| `stop_firewall.yml`         | yes       | no    | Stops and disables `firewalld`.                                                                          |
| `uptime.yml`                | yes       | yes   | Reports uptime in days and recommends a reboot when uptime exceeds 30 days.                              |

## Usage

Run a playbook against your inventory, optionally limiting to specific hosts:

```shell
# Collect /etc/hosts from every managed node (writes to ./tmp/)
mkdir -p tmp
ansible-playbook -i inventories/prod/hosts.yml download_hosts_files.yml

# Force an NTP sync on a single host
ansible-playbook -i inventories/prod/hosts.yml ntpdate.yml --limit db01

# Report uptime across the fleet
ansible-playbook -i inventories/prod/hosts.yml uptime.yml

# Reboot a maintenance group
ansible-playbook -i inventories/prod/hosts.yml reboot.yml --limit maintenance
```

## Notes

- `download_hosts_files.yml` expects a local `tmp/` directory on the control node.
- `reboot.yml` and `shutdown.yml` use a one-minute delay so that an accidental
  run can still be aborted with `shutdown -c` on the target.
- `stop_firewall.yml` disables the host firewall permanently — use with care and
  only where a network firewall provides protection.

## License

GPL-3.0-only

## Author

Thomas Bendler - [https://www.thbe.org/](https://www.thbe.org/)
