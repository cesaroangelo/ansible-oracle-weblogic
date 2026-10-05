# Changelog

## 2.0.0

Complete rewrite. Not compatible with 1.x.

### Added
- Standard role layout with `meta/argument_specs.yml` validation.
- WebLogic 12.2.1.4 / 14.1.x silent install via response file and central inventory.
- Domain creation with WLST offline, credentials passed through the environment.
- systemd units for Administration Server and Node Manager.
- firewalld integration, optional SSL and secured production mode.
- Explicit secured production mode handling (14.1.2 enables it by default with `prod`).
- `weblogic_installer_extra_args` (e.g. `-ignoreSysPrereqs`).
- CI: yamllint, ansible-lint (production profile), syntax check, Molecule on EL8 and EL9.

### Removed
- Playbooks in `defaults/`, SysV init script, WebLogic 10.3.6 / JDK 7 / EL6 support.
- Hard-coded credentials, IP addresses and project-specific settings.
- Disabling iptables.

### Fixed
- License mismatch: GPL-3.0-or-later everywhere, `LICENSE` added.
