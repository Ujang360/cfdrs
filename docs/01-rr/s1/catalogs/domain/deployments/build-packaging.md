# Deployments — Build and Packaging

> Part of the [Deployments Behavior Catalog](README.md).

## Build Artifacts

### Binary Compilation

The [Makefile](https://github.com/cloudflare/cloudflared/blob/2026.3.0/Makefile) produces the core binary via `make cloudflared`. Compile-time version information is embedded via `-ldflags`:

| Linker variable | Source | Purpose |
|---|---|---|
| `main.Version` | `git describe --tags --always` | Semantic version string |
| `main.BuildTime` | `date -u -r RELEASE_NOTES` | UTC build timestamp |
| `updater.BuiltForPackageManager` | `PACKAGE_MANAGER` env var | Identifies deb/rpm/brew origin for auto-update suppression |
| `metrics.Runtime` | `CONTAINER_BUILD` env var | Set to `virtual` in Docker builds — changes metrics bind behavior |
| `main.BuildType` | `FIPS=true` | Set to `FIPS` for FIPS-compliant builds |

Build artifact evidence: [cmd/cloudflared/cliutil/build_info](../../../atoms/cmd/cloudflared/cliutil/build_info.md), [cmd/cloudflared/main](../../../atoms/cmd/cloudflared/main.md).

### Binary Naming

| Build mode | Binary name | Notes |
|---|---|---|
| Standard | `cloudflared` | Default for all non-FIPS builds |
| FIPS | `cloudflared-fips` | Unless `ORIGINAL_NAME=true`, which keeps `cloudflared` |
| Nightly | `cloudflared-nightly` | Package name only; binary installed as `cloudflared` |

### Install Destinations

| Artifact | Makefile path | Notes |
|---|---|---|
| Binary | `/usr/bin/cloudflared` (`$(PREFIX)/bin/`) | Installed with `0755` permissions |
| Man page | `/usr/share/man/man1/cloudflared.1` (`$(PREFIX)/share/man/man1/`) | Installed with `0644` permissions |

### FIPS Compliance

FIPS builds use `-linkmode=external -extldflags=-static` with Go build tags `osusergo netgo fips`. After compilation, `check-fips.sh` validates the binary for FIPS compliance.

FIPS evidence: [fips/fips](../../../atoms/fips/fips.md), [fips/nofips](../../../atoms/fips/nofips.md).

## Packaging Formats

### Linux Packages (deb/rpm)

Both formats are produced by [fpm](https://fpm.readthedocs.io/) via the `build_package` Makefile function:

| Field | Value |
|---|---|
| Description | `Cloudflare Tunnel daemon` |
| Vendor | `Cloudflare` |
| License | `Apache License Version 2.0` |
| After-install script | `postinst.sh` |
| After-remove script | `postrm.sh` |
| Installed binary | `cloudflared` → `/usr/bin/cloudflared` |
| Installed man page | `cloudflared.1` → `/usr/share/man/man1/cloudflared.1` |

#### Post-Install Script (`postinst.sh`)

```shell
#!/bin/bash
set -eu
ln -sf /usr/bin/cloudflared /usr/local/bin/cloudflared
mkdir -p /usr/local/etc/cloudflared/
touch /usr/local/etc/cloudflared/.installedFromPackageManager || true
```

Three operations:

1. Creates a symlink `/usr/local/bin/cloudflared` → `/usr/bin/cloudflared` for PATH compatibility.
2. Ensures the config directory `/usr/local/etc/cloudflared/` exists.
3. Creates the sentinel file `.installedFromPackageManager` that disables auto-update.

#### Post-Remove Script (`postrm.sh`)

```shell
#!/bin/bash
set -eu
rm -f /usr/local/bin/cloudflared
rm -f /usr/local/etc/cloudflared/.installedFromPackageManager
```

Removes the symlink and sentinel file. Does **not** remove config directory or config files — operator data is preserved.

### Windows MSI Installer

The [cloudflared.wxs](https://github.com/cloudflare/cloudflared/blob/2026.3.0/cloudflared.wxs) WiX manifest defines:

| Property | Value |
|---|---|
| Install scope | `perMachine` |
| Install directory | `%ProgramFiles%\cloudflared\` (64-bit) or `%ProgramFiles(x86)%\cloudflared\` (32-bit) |
| PATH addition | `[INSTALLDIR]` appended to system `PATH` |
| Upgrade behavior | In-place upgrade from version `2020.8.0` onward; blocks downgrades |
| UpgradeCode GUID | `23f90fdd-9328-47ea-ab52-5380855a4b12` |

### Docker Container Image

| Property | Value |
|---|---|
| Builder base | `golang:1.24.13` |
| Runtime base | `gcr.io/distroless/base-debian13:nonroot` |
| Binary location | `/usr/local/bin/cloudflared` |
| User / Group | `65532:65532` (`nonroot`) |
| Entrypoint | `["cloudflared", "--no-autoupdate"]` |
| Default CMD | `["version"]` |
| Architectures | `amd64`, `arm64` |
| Registry | `docker.io/cloudflare/cloudflared` |
| Build env | `CGO_ENABLED=0`, `CONTAINER_BUILD=1` |

Container builds set `CONTAINER_BUILD=1`, which embeds `metrics.Runtime=virtual` — this changes the metrics server bind address behavior for containerized environments.

## Complete Filesystem Layout

### Linux (Package-Managed)

| Path | Type | Owner | Purpose |
|---|---|---|---|
| `/usr/bin/cloudflared` | binary | root | Primary binary installed by deb/rpm |
| `/usr/local/bin/cloudflared` | symlink | root | `postinst.sh` symlink to `/usr/bin/cloudflared` |
| `/usr/share/man/man1/cloudflared.1` | file | root | Man page |
| `/etc/cloudflared/config.yml` | file | root | Service config path (copied during `service install`) |
| `/usr/local/etc/cloudflared/` | directory | root | Default config search directory |
| `/usr/local/etc/cloudflared/.installedFromPackageManager` | sentinel | root | Disables auto-update |
| `/etc/systemd/system/cloudflared.service` | unit | root | Systemd service unit |
| `/etc/systemd/system/cloudflared-update.service` | unit | root | Systemd update one-shot unit |
| `/etc/systemd/system/cloudflared-update.timer` | timer | root | Daily update trigger |
| `/etc/init.d/cloudflared` | script | root | SysV init script (if not systemd) |
| `/etc/rc{2,3,4,5}.d/S50et` | symlink | root | SysV start runlevel links |
| `/etc/rc{0,1,6}.d/K02et` | symlink | root | SysV stop runlevel links |
| `/var/run/cloudflared.pid` | PID file | root | SysV PID tracking |
| `/var/log/cloudflared/` | directory | root | Default log directory |
| `/var/log/cloudflared.log` | file | root | SysV stdout log |
| `/var/log/cloudflared.err` | file | root | SysV stderr log |
| `~/.cloudflared/cert.pem` | file | user | Origin certificate (from `cloudflared login`) |
| `~/.cloudflared/<uuid>.json` | file | user | Tunnel credential file |

### macOS

| Path | Type | Owner | Purpose |
|---|---|---|---|
| Binary path | binary | user/root | Wherever installed (Homebrew: `/opt/homebrew/bin/cloudflared`) |
| `/Library/LaunchDaemons/com.cloudflare.cloudflared.plist` | plist | root | System daemon (root install) |
| `~/Library/LaunchAgents/com.cloudflare.cloudflared.plist` | plist | user | User agent (non-root install) |
| `/Library/Logs/com.cloudflare.cloudflared.out.log` | file | root | Daemon stdout log |
| `/Library/Logs/com.cloudflare.cloudflared.err.log` | file | root | Daemon stderr log |
| `~/Library/Logs/com.cloudflare.cloudflared.out.log` | file | user | User agent stdout log |
| `~/Library/Logs/com.cloudflare.cloudflared.err.log` | file | user | User agent stderr log |
| `/usr/local/etc/cloudflared/` | directory | root | Default config search directory |
| `~/.cloudflared/cert.pem` | file | user | Origin certificate |
| `~/.cloudflared/<uuid>.json` | file | user | Tunnel credential file |

### Windows

| Path | Type | Owner | Purpose |
|---|---|---|---|
| `%ProgramFiles%\cloudflared\cloudflared.exe` | binary | SYSTEM | MSI install target (64-bit) |
| `%ProgramFiles(x86)%\cloudflared\cloudflared.exe` | binary | SYSTEM | MSI install target (32-bit) |
| `%USERPROFILE%\.cloudflared\cert.pem` | file | user | Origin certificate |
| `%USERPROFILE%\.cloudflared\<uuid>.json` | file | user | Tunnel credential file |
| `%USERPROFILE%\.cloudflared\config.yml` | file | user | User config file |
| Windows Event Log (`Cloudflared`) | log | SYSTEM | Event log registered at service install |
| System PATH | env var | SYSTEM | MSI appends install directory to system PATH |

Windows config directory discovery: `CFDPATH` env var → `%ProgramFiles(x86)%\cloudflared` → `%ProgramFiles%\cloudflared`.

### Docker Container

| Path | Type | Owner | Purpose |
|---|---|---|---|
| `/usr/local/bin/cloudflared` | binary | `65532:65532` | Copied from builder stage with `--chown=nonroot` |

No config files, credential files, or service units are embedded in the image. All configuration must be provided at runtime via volume mounts, environment variables, or CLI arguments (commonly `--token`).
