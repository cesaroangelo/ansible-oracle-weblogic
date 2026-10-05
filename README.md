# Ansible Role: weblogic

[![CI](https://github.com/cesaroangelo/ansible-oracle-weblogic/actions/workflows/ci.yml/badge.svg)](https://github.com/cesaroangelo/ansible-oracle-weblogic/actions/workflows/ci.yml)

Installs Oracle WebLogic Server in silent mode, creates a domain with WLST offline and runs the
Administration Server and the per-domain Node Manager as systemd services.

- Idempotent: installation and domain creation run only once.
- Unprivileged: the installer and WLST run as `weblogic_user`.
- Credentials are passed to WLST through the environment and never logged.
- Input validated through `meta/argument_specs.yml`.

## Requirements

| Component      | Supported                                            |
|----------------|------------------------------------------------------|
| Ansible        | ansible-core >= 2.16 (EL8 targets: 2.16, see below)  |
| Collections    | `ansible.posix` (see `requirements.yml`)             |
| Target OS      | RHEL / Rocky / AlmaLinux / Oracle Linux 8, 9         |
| WebLogic / JDK | 14.1.2 (JDK 17, 21), 14.1.1 (JDK 8, 11), 12.2.1.4 (JDK 8) |

Oracle media is licensed and must be downloaded from [Oracle eDelivery](https://edelivery.oracle.com)
or My Oracle Support:

- WebLogic generic installer: the `.zip` as downloaded, or the extracted `.jar`.
- A certified JDK `.tar.gz`, or an existing JDK on the host.

```sh
ansible-galaxy collection install -r requirements.yml
```

EL8 ships Python 3.6. ansible-core 2.17+ no longer supports it on targets, and the `dnf`
module cannot use the EL8 dnf bindings from a newer Python. Manage EL8 hosts with ansible-core 2.16.

## Role variables

Required:

| Variable                  | Description                                                  |
|---------------------------|--------------------------------------------------------------|
| `weblogic_installer`      | Installer `.jar` or `.zip`, on the controller unless `weblogic_installer_remote_src: true` |
| `weblogic_admin_password` | Administrator password. Keep it in ansible-vault             |

Main defaults (full list in [`defaults/main.yml`](defaults/main.yml) and
[`meta/argument_specs.yml`](meta/argument_specs.yml)):

| Variable                       | Default                                          |
|--------------------------------|--------------------------------------------------|
| `weblogic_user` / `weblogic_group` | `oracle` / `oinstall`                        |
| `weblogic_oracle_home`         | `/u01/app/oracle/product/wls`                    |
| `weblogic_java_home`           | `/u01/app/oracle/product/jdk`                    |
| `weblogic_jdk_archive`         | `""` (use the JDK already in `weblogic_java_home`) |
| `weblogic_domain_name`         | `base_domain`                                    |
| `weblogic_domain_home`         | `/u01/app/oracle/config/domains/<domain>`        |
| `weblogic_server_start_mode`   | `prod` (`dev`, `prod`, `secure`)                 |
| `weblogic_administration_port` | `9002` (`secure` mode)                           |
| `weblogic_installer_extra_args`| `[]`, e.g. `["-ignoreSysPrereqs"]`               |
| `weblogic_admin_username`      | `weblogic`                                       |
| `weblogic_admin_listen_port`   | `7001`                                           |
| `weblogic_admin_ssl_enabled`   | `false` (port `7002`)                            |
| `weblogic_nodemanager_enabled` | `true` (`localhost:5556`)                        |
| `weblogic_service_state`       | `started`                                        |
| `weblogic_firewall_manage`     | `true` (only if firewalld is running)            |

Domain modes:

- `prod`: production mode. Secured production mode, enabled by default with production mode
  since 14.1.2, is explicitly disabled so the plain listen port stays available.
- `secure`: secured production mode (14.1.1+). The plain port is disabled; the Administration
  Server listens on `weblogic_admin_ssl_listen_port` and `weblogic_administration_port`.

## Example

```yaml
# group_vars/weblogic/main.yml
weblogic_jdk_archive: files/jdk-21_linux-x64_bin.tar.gz
weblogic_installer: files/V1045135-01.zip
weblogic_domain_name: app_domain
weblogic_admin_password: "{{ vault_weblogic_admin_password }}"
```

```yaml
# site.yml
- name: Deploy WebLogic Server
  hosts: weblogic
  become: true
  roles:
    - role: cesaroangelo.weblogic
```

A complete example is in [`examples/`](examples/).

## Tags

`weblogic_prerequisites`, `weblogic_jdk`, `weblogic_install`, `weblogic_domain`, `weblogic_service`.

## Services

| Unit                                     | Purpose              |
|------------------------------------------|----------------------|
| `weblogic-<domain>-adminserver.service`  | Administration Server |
| `weblogic-<domain>-nodemanager.service`  | Node Manager         |

Logs go to the journal: `journalctl -u weblogic-<domain>-adminserver`.

## Notes

- `boot.properties` is created once and then encrypted by WebLogic. To rotate the admin
  password, change it in WebLogic first, then remove `boot.properties` and rerun the role.
- Domain creation is skipped when `config/config.xml` already exists: later changes to
  domain variables are not applied to an existing domain.

## Testing

```sh
pip install ansible-lint yamllint molecule "molecule-plugins[docker]" docker
ansible-galaxy collection install -r requirements.yml community.docker
yamllint . && ansible-lint
molecule test --platform-name el9
```

The Molecule scenario runs the role on Rocky Linux 8 (`secure` mode, no Node Manager) and 9
(`prod` mode), checks idempotence and verifies services, ports and file permissions. Oracle
media cannot be redistributed, so the scenario uses a test double for the installer and WLST.

## License

GPL-3.0-or-later

## Author

Angelo Cesaro
