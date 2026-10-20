# sysaudit

A read-only Linux system health audit for Bash. Prints a plain-text
report to stdout covering OS, CPU, memory, disk usage, pending package
updates, listening TCP ports, recent failed SSH logins, and key
service states — in a single command.

## Problem

Routine server health checks are scattered across a dozen commands:
`df`, `free`, `uptime`, `apt-get -s upgrade`, `ss`, `journalctl`…
Running them by hand is slow, inconsistent, and easy to forget.
`sysaudit` collects the essentials into one read-only report that can
be pasted into a ticket, saved for a baseline, or run on a schedule.

## Intended users

- Linux system administrators and DevOps engineers
- Managed-service providers doing fleet health checks
- Anyone who wants a quick, non-invasive snapshot of a Linux host

## Install

No dependencies beyond common Linux tools. Download and make it executable:

```sh
curl -O https://raw.githubusercontent.com/sysnetsol/sysaudit/main/sysaudit
chmod +x sysaudit
./sysaudit
```

Works as a non-root user; some sections (SSH log scan) show more detail
when run as root.

## Usage

```
sysaudit [--help] [--version]

Prints a plain-text report covering:
  * OS, kernel, hostname, uptime
  * CPU count and load averages
  * Memory and swap usage
  * Disk usage (warning at >= 80%)
  * Pending package updates (apt-get -s / dnf check-update; dry-run only)
  * Listening TCP ports
  * Recent failed SSH login attempts
  * State of common services (sshd, cron, networking, ...)

The script is read-only: it never installs anything or changes system state.
Some sections may be limited when run as a non-root user.

Exit status: 0 if the report completed, 2 on usage error (unknown option).
```

## Example output

Real output from a test run on Ubuntu 24.04.5 LTS (2026-10-18):

```
sysaudit 1.0.0 — system health report
generated: 2026-10-18T13:10:16Z

=== System ===
hostname:              server01
os:                    Ubuntu 24.04.5 LTS
kernel:                7.0.0-39-generic
uptime:                up 1 hour, 36 minutes

=== CPU ===
logical CPUs:          2
load average:          2.39 1.01 0.43
model:                 AMD EPYC 9D25 126-Core Processor

=== Memory ===
memory:                7935 MB total, 6238 MB used, 1697 MB available (78% used)
swap:                  0 MB total, 0 MB used

=== Disk usage (warning at >= 80%) ===
overlay on /:          1% full (72M used of 7.5G)
/dev/mapper/rv on /home/user: 1% full (236M used of 100G)

=== Pending updates (dry-run only — nothing is installed) ===
upgradable:            0 package(s)

=== Listening TCP ports ===
no listening TCP ports found (or permission denied)

=== Failed SSH logins (recent, read-only log scan) ===
journalctl (ssh/sshd): 0 matching event(s) in last 24h

=== Key service states (systemctl, tolerated when unavailable) ===
sshd:                  inactive
ssh:                   inactive
cron:                  inactive
systemd-resolved:      inactive
NetworkManager:        inactive
networking:            inactive
docker:                inactive

--- end of report ---
```

## Features

- **One command, full picture** — nine sections in a single readable report
- **Read-only by design** — package checks use `apt-get -s` (simulate) or
  `dnf check-update` only; nothing is ever installed or changed
- **Graceful degradation** — missing tools, denied permissions, and
  non-systemd systems produce clear notes instead of errors
- **Warning thresholds** — filesystems at >= 80% are flagged
- **Log-scan tolerant** — tries `journalctl`, falls back to
  `/var/log/auth.log`, explains itself when neither is readable
- **Zero dependencies** — pure Bash

## Limitations

- A snapshot, not a monitor — for alerting, pair with real monitoring
- Pending-update simulation uses cached apt metadata (it does not refresh
  package lists); on very locked-down
  systems it may need root to be accurate
- The SSH-login scan only covers what the running user can read
- Container environments may show no listening ports and inactive services
  (as in the example above — that is normal for a container)

## Supported platforms

Tested on **Ubuntu 24.04.5 LTS**. Should work on any modern Linux
distribution (Debian/Ubuntu, RHEL/Fedora/CentOS, Arch, Alpine) with
Bash. `dnf`-based systems use `dnf check-update`; `apt`-based systems
use `apt-get -s upgrade`.

## Security notes

- **Read-only**: the script never writes files, starts/stops services, or
  installs updates. The only package-manager invocation is
  `apt-get -s upgrade` (simulation) or `dnf check-update`.
- **No data exfiltration**: the script makes no network requests at all.
  The report goes to stdout and stays on the machine.
- No credentials are read, written, or transmitted. Log lines shown in
  the SSH section may contain usernames from failed logins — handle the
  report accordingly if you share it.
- Safe to run as a non-root user; root adds detail to the SSH-log section.

## Contributing

Issues and pull requests are welcome. Please keep changes Bash-only,
read-only, and graceful about missing tools. Test on at least one real
Linux system before submitting.

## License

MIT License — see [LICENSE](./LICENSE).
