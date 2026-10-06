# Ansible Role: weblogic

[![CI](https://github.com/cesaroangelo/ansible-oracle-weblogic/actions/workflows/ci.yml/badge.svg)](https://github.com/cesaroangelo/ansible-oracle-weblogic/actions/workflows/ci.yml)

Installs Oracle WebLogic Server, creates a domain with WLST offline and runs the Administration
Server and Node Manager under systemd. Idempotent, runs as an unprivileged user, never logs credentials.

## Requirements

- ansible-core >= 2.16 (EL8 targets: 2.16 only, EL8 ships Python 3.6)
- `ansible-galaxy collection install -r requirements.yml`
- RHEL / Rocky / AlmaLinux / Oracle Linux 8, 9
- WebLogic 14.1.2 (JDK 17/21), 14.1.1 (JDK 8/11), 12.2.1.4 (JDK 8)
- Oracle installer (`.zip` or `.jar`) and JDK `.tar.gz` from [Oracle eDelivery](https://edelivery.oracle.com)

## Variables

Required: `weblogic_installer`, `weblogic_admin_password` (use ansible-vault).

| Variable | Default |
|---|---|
| `weblogic_jdk_archive` | `""` (use existing `weblogic_java_home`) |
| `weblogic_domain_name` | `base_domain` |
| `weblogic_server_start_mode` | `prod` (`dev`, `prod`, `secure`) |
| `weblogic_admin_listen_port` | `7001` |
| `weblogic_nodemanager_enabled` | `true` |
| `weblogic_installer_extra_args` | `[]` (e.g. `-ignoreSysPrereqs`) |
| `weblogic_firewall_manage` | `true` |

Full list: [`defaults/main.yml`](defaults/main.yml), [`meta/argument_specs.yml`](meta/argument_specs.yml).

`prod` disables secured production mode (default since 14.1.2). `secure` enables it: SSL
(`7002`) and administration port (`9002`) only.

## Example

```yaml
- name: Deploy WebLogic Server
  hosts: weblogic
  become: true
  vars:
    weblogic_jdk_archive: files/jdk-21_linux-x64_bin.tar.gz
    weblogic_installer: files/V1045135-01.zip
    weblogic_admin_password: "{{ vault_weblogic_admin_password }}"
  roles:
    - cesaroangelo.weblogic
```

Services: `weblogic-<domain>-adminserver`, `weblogic-<domain>-nodemanager`.
Tags: `weblogic_prerequisites`, `weblogic_jdk`, `weblogic_install`, `weblogic_domain`, `weblogic_service`.

## Notes

- The domain is created once; later variable changes do not reconfigure it.
- `boot.properties` is written once. To rotate the password, change it in WebLogic, delete the file, rerun.

## Testing

```sh
yamllint . && ansible-lint
molecule test --platform-name el9   # el8 needs ansible-core 2.16
```

## License

Copyright (C) 2015-2026 Angelo Cesaro <cesaro.angelo@gmail.com>

Licensed under the GNU General Public License v3.0 only. See [LICENSE](LICENSE).
