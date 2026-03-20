# Features — Operator and Platform Stakeholders

> Part of the [Features Behavior Catalog](README.md).

## Stakeholder 1 — Operator / Human

The operator interacts with cloudflared through the CLI command tree, YAML/JSON configuration files, and platform service management commands. This stakeholder initiates all tunnel lifecycle operations and controls runtime behavior through flags and config.

### CLI Command Surface

The top-level command tree provides tunnel CRUD, access login, service management, and diagnostic commands.

| Command family | Description | Key atoms |
|---|---|---|
| `tunnel` (run, create, delete, list, info, cleanup, token) | Named-tunnel lifecycle and execution. | [cmd/cloudflared/tunnel/cmd](../../../atoms/cmd/cloudflared/tunnel/cmd.md), [cmd/cloudflared/tunnel/subcommands](../../../atoms/cmd/cloudflared/tunnel/subcommands.md), [cmd/cloudflared/tunnel/subcommand_context](../../../atoms/cmd/cloudflared/tunnel/subcommand_context.md) |
| `tunnel route dns`, `tunnel route ip` | DNS and IP route provisioning for tunnels. | [cmd/cloudflared/tunnel/teamnet_subcommands](../../../atoms/cmd/cloudflared/tunnel/teamnet_subcommands.md), [cmd/cloudflared/tunnel/subcommand_context_teamnet](../../../atoms/cmd/cloudflared/tunnel/subcommand_context_teamnet.md) |
| `tunnel route vnet` | Virtual-network CRUD for private routing. | [cmd/cloudflared/tunnel/vnets_subcommands](../../../atoms/cmd/cloudflared/tunnel/vnets_subcommands.md), [cmd/cloudflared/tunnel/subcommand_context_vnets](../../../atoms/cmd/cloudflared/tunnel/subcommand_context_vnets.md) |
| `access` (login, curl, ssh, rdp, smb, tcp) | Cloudflare Access login flow and application-layer tunneling. | [cmd/cloudflared/access/cmd](../../../atoms/cmd/cloudflared/access/cmd.md), [cmd/cloudflared/access/carrier](../../../atoms/cmd/cloudflared/access/carrier.md) |
| `service install/uninstall` | Platform service lifecycle (systemd, launchd, SCM). | [cmd/cloudflared/linux_service](../../../atoms/cmd/cloudflared/linux_service.md), [cmd/cloudflared/macos_service](../../../atoms/cmd/cloudflared/macos_service.md), [cmd/cloudflared/windows_service](../../../atoms/cmd/cloudflared/windows_service.md) |
| `tail` | Remote log streaming via management service. | [cmd/cloudflared/tail/cmd](../../../atoms/cmd/cloudflared/tail/cmd.md) |
| `version`, `update` | Build-info display and manual update trigger. | [cmd/cloudflared/cliutil/build_info](../../../atoms/cmd/cloudflared/cliutil/build_info.md), [cmd/cloudflared/updater/update](../../../atoms/cmd/cloudflared/updater/update.md) |

Deeper CLI and flag documentation: [cli](../cli.md).

### Configuration Contract

The operator supplies runtime configuration through YAML files discovered by a platform-dependent search path, environment variables, or CLI flags. Configuration undergoes a layered merge: CLI flags override config-file values which override defaults.

| Aspect | Contract | Atoms |
|---|---|---|
| Config file discovery | Platform search path → `FindDefaultConfigPath()` | [config/configuration](../../../atoms/config/configuration.md) |
| Config model types | `Configuration`, `NamedTunnel`, `OriginRequestConfig` | [config/model](../../../atoms/config/model.md) |
| Ingress rules | `ingress:` YAML block → rule matching engine | [ingress/ingress](../../../atoms/ingress/ingress.md), [ingress/rule](../../../atoms/ingress/rule.md) |
| Credential discovery | origin cert search → tunnel credential file → token flag | [credentials/credentials](../../../atoms/credentials/credentials.md), [cmd/cloudflared/tunnel/credential_finder](../../../atoms/cmd/cloudflared/tunnel/credential_finder.md) |
| Flag definitions | All `--flag` definitions and env-var bindings | [cmd/cloudflared/flags/flags](../../../atoms/cmd/cloudflared/flags/flags.md), [cmd/cloudflared/tunnel/configuration](../../../atoms/cmd/cloudflared/tunnel/configuration.md) |

Deeper configuration documentation: [config](../config.md).

### Quick Tunnel Mode

When no named tunnel is configured, the operator can run an ephemeral quick tunnel via `--url` or `--hello-world`. Quick tunnels obtain a random `*.trycloudflare.com` hostname from the quick-tunnel service endpoint.

| Contract | Detail | Atom |
|---|---|---|
| Provisioning endpoint | `https://api.trycloudflare.com` POST | [cmd/cloudflared/tunnel/quick_tunnel](../../../atoms/cmd/cloudflared/tunnel/quick_tunnel.md) |
| Constraints | No ICMP packet routing, no custom ingress rules | [cmd/cloudflared/tunnel/cmd](../../../atoms/cmd/cloudflared/tunnel/cmd.md) |
| URL broadcasting | `observer.SendURL(quickTunnelURL)` → metrics counter | [connection/observer](../../../atoms/connection/observer.md) |

### Stdin Control

An undocumented operator interface allows sending commands via stdin when `--stdin-control` is set. The only supported command is `reconnect [delay]`, which sends a `ReconnectSignal` to force reconnection of a random tunnel connection.

Primary evidence: [supervisor/supervisor](../../../atoms/supervisor/supervisor.md), [supervisor/external_control](../../../atoms/supervisor/external_control.md).

## Stakeholder 8 — OS / Platform

The operating system interacts with cloudflared through service lifecycle management, signal handling, auto-update mechanisms, and build-time feature gating.

### Service Lifecycle

Platform-specific service installation writes unit files, plist files, or Windows service registrations.

| Platform | Service manager | Install atom | Template atom |
|---|---|---|---|
| Linux | systemd / sysv | [cmd/cloudflared/linux_service](../../../atoms/cmd/cloudflared/linux_service.md) | [cmd/cloudflared/service_template](../../../atoms/cmd/cloudflared/service_template.md) |
| macOS | launchd (daemon + user agent) | [cmd/cloudflared/macos_service](../../../atoms/cmd/cloudflared/macos_service.md) | [cmd/cloudflared/service_template](../../../atoms/cmd/cloudflared/service_template.md) |
| Windows | Service Control Manager | [cmd/cloudflared/windows_service](../../../atoms/cmd/cloudflared/windows_service.md) | [cmd/cloudflared/service_template](../../../atoms/cmd/cloudflared/service_template.md) |

Common service behavior: [cmd/cloudflared/common_service](../../../atoms/cmd/cloudflared/common_service.md), [cmd/cloudflared/generic_service](../../../atoms/cmd/cloudflared/generic_service.md).

Application entry points: [cmd/cloudflared/app_service](../../../atoms/cmd/cloudflared/app_service.md), [cmd/cloudflared/app_forward_service](../../../atoms/cmd/cloudflared/app_forward_service.md).

Deeper platform documentation: [platforms](../platforms.md), [deployments](../deployments/README.md).

### Signal Handling

cloudflared registers signal handlers for graceful shutdown and reconnection.

| Signal | Behavior | Atoms |
|---|---|---|
| `SIGINT`, `SIGTERM` | Initiates graceful shutdown → grace period → force exit | [signal/safe_signal](../../../atoms/signal/safe_signal.md) |
| systemd `READY=1` | Sent after first successful connection via `daemon.SdNotify` | [cmd/cloudflared/tunnel/cmd](../../../atoms/cmd/cloudflared/tunnel/cmd.md) |

### Auto-Update Contract

The auto-updater runs as a background goroutine checking for new versions at configurable intervals.

| Aspect | Detail | Atoms |
|---|---|---|
| Update check | Periodic version comparison against update server | [cmd/cloudflared/updater/check](../../../atoms/cmd/cloudflared/updater/check.md) |
| Update execution | Download, verify, exec replacement binary | [cmd/cloudflared/updater/update](../../../atoms/cmd/cloudflared/updater/update.md) |
| Service integration | Update-as-service for managed installs | [cmd/cloudflared/updater/service](../../../atoms/cmd/cloudflared/updater/service.md) |
| Workers update | Alternative update path for Workers-based deployment | [cmd/cloudflared/updater/workers_update](../../../atoms/cmd/cloudflared/updater/workers_update.md), [cmd/cloudflared/updater/workers_service](../../../atoms/cmd/cloudflared/updater/workers_service.md) |

### FIPS Build-Tag Gating

The FIPS build tag controls whether the binary uses FIPS-compliant cryptographic libraries. This is a compile-time feature, not a runtime flag.

Primary evidence: [fips/fips](../../../atoms/fips/fips.md), [fips/nofips](../../../atoms/fips/nofips.md).

### Process Supervision

The overwatch layer manages the top-level application lifecycle including restart behavior.

Primary evidence: [overwatch/manager](../../../atoms/overwatch/manager.md), [overwatch/app_manager](../../../atoms/overwatch/app_manager.md).
