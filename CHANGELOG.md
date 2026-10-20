# Changelog

All notable changes to this project will be documented in this file.

## [1.0.0] - 2026-10-18

Initial release.

- Read-only system health report: OS/kernel/hostname/uptime, CPU load,
  memory and swap, disk usage with 80% warn threshold, pending updates
  (apt-get -s / dnf check-update, dry-run only), listening TCP ports,
  failed SSH logins (journalctl with /var/log/auth.log fallback), and
  key service states
- Tolerates missing tools, denied permissions, and non-systemd systems
- `--help` and `--version` flags; plain-text report on stdout
