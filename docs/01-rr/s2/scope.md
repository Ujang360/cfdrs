# S2.2 - Scope

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| S2.2 status | Closed |
| Consumes | [dependency-decisions](dependency-decisions.md) Section 2.2 entry conditions |
| RR target | Linux x86-64, systemd only |
| Version floors | kernel >= 5.10, glibc >= 2.31, systemd >= 247 |

This document fixes the RR scope boundary before any S3 work starts. It
turns the stakeholder feature catalog, platform catalog, and porting-friction
catalog into an executable scoping contract for the Rust rewrite.

**Scope drivers:**

- Tier 1 runtime target is Linux x86-64 on systemd, including bare-host,
  Docker, and Kubernetes deployment shapes.
- The transport core is QUIC-only for RR. The HTTP/2 tunnel path is
  intentionally deferred.
- Must / Should / Could / Won't follows the user-confirmed rule set:
  Must = required for a QUIC tunnel to register, proxy a request, and shut
  down cleanly on Linux/systemd; Should = present in normal Go operation but
  not required for the minimal RR binary purpose; Could = rare or
  quality-of-life behavior; Won't = deferred beyond S4 or excluded.
- The [Features catalog](../s1/catalogs/cross-cutting/features/README.md)
  contributes 127 unique stakeholder-scoped atoms. Every unique atom is
  classified exactly once below.
- The [Platforms catalog](../s1/catalogs/domain/platforms.md) contributes 31
  platform-scoped atoms. Every atom is mapped to Tier 1 or FC-deferred below.

**S2.2 exit gate:** Every feature in Must/Should/Could remains traceable to
at least one S1 atom or S2.1 dependency decision. Every Won't decision is
anchored either to S2.1 or to an explicit S1 behavioral finding. The platform
matrix must stay coherent with Layer 9 of [dependency-decisions](dependency-decisions.md).

## Platform Matrix

### Runtime Envelope

| Dimension | RR decision | Evidence |
| --- | --- | --- |
| Operating system | Linux only | [platform-substrates](../s1/catalogs/cross-cutting/platform-substrates.md), [platforms](../s1/catalogs/domain/platforms.md) |
| CPU architecture | x86-64-v2 baseline, x86-64-v4 optimization target | [platform-substrates](../s1/catalogs/cross-cutting/platform-substrates.md), [dependency-decisions](dependency-decisions.md) |
| Init system | systemd only, `Type=notify`, `READY=1` and `STOPPING=1` only | [platform-substrates](../s1/catalogs/cross-cutting/platform-substrates.md), [platforms](../s1/catalogs/domain/platforms.md), [dependency-decisions](dependency-decisions.md) |
| Deployment shapes | bare host, Docker, Kubernetes | [platform-substrates](../s1/catalogs/cross-cutting/platform-substrates.md), [platforms](../s1/catalogs/domain/platforms.md) |
| NUMA model | none; single-socket target | [dependency-decisions](dependency-decisions.md) |
| Minimum kernel | 5.10 LTS floor; set now to avoid backsliding while `io_uring` stays deferred | [dependency-decisions](dependency-decisions.md) |
| Tunnel transport | QUIC only | [edge-and-access](../s1/catalogs/cross-cutting/features/edge-and-access.md), [dependency-decisions](dependency-decisions.md) |
| Crypto mode | non-FIPS only | [operator-and-platform](../s1/catalogs/cross-cutting/features/operator-and-platform.md), [platform-substrates](../s1/catalogs/cross-cutting/platform-substrates.md) |

### Version Floor Rationale

The three version floors (kernel ≥ 5.10, glibc ≥ 2.31, systemd ≥ 247) are
chosen as a coherent set: they correspond to **Debian 11 Bullseye**, the
oldest actively-supported Debian release that satisfies all three
simultaneously.

| Component | Floor | Why this version |
| --- | --- | --- |
| kernel | 5.10 | Kernel.org LTS (Dec 2020). Provides mature epoll, stable QUIC UDP socket options, and a forward-compatible floor for `io_uring` evaluation post-S4 without requiring a floor bump. Matches the Debian 11 stock kernel. |
| glibc | 2.31 | Ships with Debian 11 and Ubuntu 20.04 LTS. This is the oldest glibc that Rust's `x86_64-unknown-linux-gnu` tier 1 target reliably links against in current toolchains. Setting this floor avoids musl cross-compilation overhead during S3. |
| systemd | 247 | Ships with Debian 11. Provides stable `sd_notify(3)` support for `READY=1` and `STOPPING=1` — the only two signals RR emits. Earlier versions (e.g., systemd 245 on Ubuntu 20.04 LTS) support the same signals, but 247 is the floor that aligns with the kernel and glibc choices above. |

Distributions that meet all three floors: Debian 11+, Ubuntu 22.04 LTS+,
Fedora 33+, RHEL 9+. Ubuntu 20.04 LTS falls below the kernel and systemd
floors and is not a supported RR target.

### Excluded Platforms

| Platform | Status | Scope note |
| --- | --- | --- |
| macOS (`darwin`) | FC-deferred | launchd, browser dispatch, diagnostics, and ICMP branches remain outside RR. |
| Windows | FC-deferred | SCM, QUIC socket params, browser dispatch, diagnostics, and ICMP branches remain outside RR. |
| FreeBSD | FC-deferred | `token/launch_browser_unix` covers browser opening, but service management and ICMP have no dedicated RR path. |

The macOS and Windows exclusions remove the following Go surface area from RR:

- [cmd/cloudflared/macos_service](../s1/atoms/cmd/cloudflared/macos_service.md)
- [cmd/cloudflared/windows_service](../s1/atoms/cmd/cloudflared/windows_service.md)
- [ingress/icmp_darwin](../s1/atoms/ingress/icmp_darwin.md)
- [ingress/icmp_windows](../s1/atoms/ingress/icmp_windows.md)
- [token/launch_browser_darwin](../s1/atoms/token/launch_browser_darwin.md)
- [token/launch_browser_windows](../s1/atoms/token/launch_browser_windows.md)
- [quic/param_windows](../s1/atoms/quic/param_windows.md)
- [diagnostic/system_collector_macos](../s1/atoms/diagnostic/system_collector_macos.md)
- [diagnostic/system_collector_windows](../s1/atoms/diagnostic/system_collector_windows.md)
- [diagnostic/network/collector_windows](../s1/atoms/diagnostic/network/collector_windows.md)

### Stage 3.9 Impact

Under this scope decision, S3.9 collapses to systemd service handling plus an
optional FIPS flag boundary. launchd, Windows SCM, and non-Linux platform work
stay outside RR. Docker and Kubernetes diagnostics remain in scope because they
are part of the Linux deployment envelope, not separate service-lifecycle
targets.

### Platform Atoms

The table below uses the audited 31-atom inventory from
[platforms](../s1/catalogs/domain/platforms.md). `Tier 1` means implemented
inside RR. `FC-deferred` means intentionally left to the follow-on phase.

#### Service Platform Adapters

| Atom | Status | Scope note |
| --- | --- | --- |
| [cmd/cloudflared/app_forward_service](../s1/atoms/cmd/cloudflared/app_forward_service.md) | Tier 1 | Needed for Linux foreground and service entry orchestration. |
| [cmd/cloudflared/app_service](../s1/atoms/cmd/cloudflared/app_service.md) | Tier 1 | Needed for Linux service entry orchestration. |
| [cmd/cloudflared/common_service](../s1/atoms/cmd/cloudflared/common_service.md) | Tier 1 | Shared Linux service argument and template plumbing remains in scope. |
| [cmd/cloudflared/generic_service](../s1/atoms/cmd/cloudflared/generic_service.md) | FC-deferred | Fallback service manager is not needed once RR commits to Linux systemd only. |
| [cmd/cloudflared/linux_service](../s1/atoms/cmd/cloudflared/linux_service.md) | Tier 1 | Linux service install and runtime control stay in scope, but only the systemd branch. |
| [cmd/cloudflared/macos_service](../s1/atoms/cmd/cloudflared/macos_service.md) | FC-deferred | macOS launchd is outside the RR platform boundary. |
| [cmd/cloudflared/windows_service](../s1/atoms/cmd/cloudflared/windows_service.md) | FC-deferred | Windows SCM is outside the RR platform boundary. |

#### Container And Host Diagnostics

| Atom | Status | Scope note |
| --- | --- | --- |
| [diagnostic/diagnostic_utils](../s1/atoms/diagnostic/diagnostic_utils.md) | Tier 1 | Shared formatting and collection helpers are needed for Linux diagnostics. |
| [diagnostic/log_collector_docker](../s1/atoms/diagnostic/log_collector_docker.md) | Tier 1 | Docker log collection is in scope for Linux container deployments. |
| [diagnostic/log_collector_host](../s1/atoms/diagnostic/log_collector_host.md) | Tier 1 | Host log collection is in scope, but only the systemd branch. |
| [diagnostic/log_collector_kubernetes](../s1/atoms/diagnostic/log_collector_kubernetes.md) | Tier 1 | Kubernetes log collection is in scope for Linux container deployments. |
| [diagnostic/log_collector_utils](../s1/atoms/diagnostic/log_collector_utils.md) | Tier 1 | Shared runtime-dispatch helpers remain in scope. |
| [diagnostic/system_collector](../s1/atoms/diagnostic/system_collector.md) | Tier 1 | Shared system collection contract remains in scope. |
| [diagnostic/system_collector_linux](../s1/atoms/diagnostic/system_collector_linux.md) | Tier 1 | Linux collector is in scope. |
| [diagnostic/system_collector_macos](../s1/atoms/diagnostic/system_collector_macos.md) | FC-deferred | macOS collector is outside the RR platform boundary. |
| [diagnostic/system_collector_utils](../s1/atoms/diagnostic/system_collector_utils.md) | Tier 1 | Shared parsing and normalization helpers stay in scope. |
| [diagnostic/system_collector_windows](../s1/atoms/diagnostic/system_collector_windows.md) | FC-deferred | Windows collector is outside the RR platform boundary. |

#### Platform Network Diagnostics

| Atom | Status | Scope note |
| --- | --- | --- |
| [diagnostic/network/collector_unix](../s1/atoms/diagnostic/network/collector_unix.md) | Tier 1 | Linux traceroute collection remains in scope through the Unix branch. |
| [diagnostic/network/collector_utils](../s1/atoms/diagnostic/network/collector_utils.md) | Tier 1 | Shared decode helpers remain in scope. |
| [diagnostic/network/collector_windows](../s1/atoms/diagnostic/network/collector_windows.md) | FC-deferred | Windows tracert handling is outside the RR platform boundary. |

#### ICMP Platform Runtime

| Atom | Status | Scope note |
| --- | --- | --- |
| [ingress/icmp_darwin](../s1/atoms/ingress/icmp_darwin.md) | FC-deferred | Darwin ICMP path is outside the RR platform boundary. |
| [ingress/icmp_generic](../s1/atoms/ingress/icmp_generic.md) | FC-deferred | Unsupported-platform fallback is not needed once RR is Linux-only. |
| [ingress/icmp_linux](../s1/atoms/ingress/icmp_linux.md) | Tier 1 | Linux ICMP path remains in scope. |
| [ingress/icmp_metrics](../s1/atoms/ingress/icmp_metrics.md) | Tier 1 | ICMP metrics remain in scope because Linux ICMP remains in scope. |
| [ingress/icmp_posix](../s1/atoms/ingress/icmp_posix.md) | Tier 1 | Linux shared POSIX ICMP helpers remain in scope. |
| [ingress/icmp_windows](../s1/atoms/ingress/icmp_windows.md) | FC-deferred | Windows ICMP path is outside the RR platform boundary. |

#### QUIC Platform Parameters

| Atom | Status | Scope note |
| --- | --- | --- |
| [quic/param_unix](../s1/atoms/quic/param_unix.md) | Tier 1 | Linux QUIC socket tuning remains in scope through the Unix branch. |
| [quic/param_windows](../s1/atoms/quic/param_windows.md) | FC-deferred | Windows QUIC socket tuning is outside the RR platform boundary. |

#### Browser Launch Dispatch

| Atom | Status | Scope note |
| --- | --- | --- |
| [token/launch_browser_darwin](../s1/atoms/token/launch_browser_darwin.md) | FC-deferred | macOS browser dispatch is outside the RR platform boundary. |
| [token/launch_browser_unix](../s1/atoms/token/launch_browser_unix.md) | Tier 1 | Linux browser dispatch remains in scope for Access login. |
| [token/launch_browser_windows](../s1/atoms/token/launch_browser_windows.md) | FC-deferred | Windows browser dispatch is outside the RR platform boundary. |

### Gating Mechanism Decisions

The table below covers every gated source surface documented in
[platform-substrates](../s1/catalogs/cross-cutting/platform-substrates.md).

#### Compile-Time Gating

| Module / atom | RR decision | Scope note |
| --- | --- | --- |
| `ingress` / [icmp_linux](../s1/atoms/ingress/icmp_linux.md) | Implement | Linux ICMP path is in scope. |
| `ingress` / [icmp_darwin](../s1/atoms/ingress/icmp_darwin.md) | Defer | Darwin path moves to FC. |
| `ingress` / [icmp_windows](../s1/atoms/ingress/icmp_windows.md) | Defer | Windows path moves to FC. |
| `ingress` / [icmp_posix](../s1/atoms/ingress/icmp_posix.md) | Implement | Linux shared POSIX helpers are needed. |
| `ingress` / [icmp_generic](../s1/atoms/ingress/icmp_generic.md) | Defer | Unsupported-platform fallback is not part of Linux-only RR. |
| `diagnostic` / [system_collector_linux](../s1/atoms/diagnostic/system_collector_linux.md) | Implement | Linux diagnostics stay in scope. |
| `diagnostic` / [system_collector_macos](../s1/atoms/diagnostic/system_collector_macos.md) | Defer | macOS diagnostics move to FC. |
| `diagnostic` / [system_collector_windows](../s1/atoms/diagnostic/system_collector_windows.md) | Defer | Windows diagnostics move to FC. |
| `diagnostic` / [collector_unix](../s1/atoms/diagnostic/network/collector_unix.md) | Implement | Linux traceroute path stays in scope. |
| `diagnostic` / [collector_windows](../s1/atoms/diagnostic/network/collector_windows.md) | Defer | Windows tracert path moves to FC. |
| `cmd` / [macos_service](../s1/atoms/cmd/cloudflared/macos_service.md) | Defer | launchd support is out of scope. |
| `cmd` / [windows_service](../s1/atoms/cmd/cloudflared/windows_service.md) | Defer | SCM support is out of scope. |
| `token` / [launch_browser_windows](../s1/atoms/token/launch_browser_windows.md) | Defer | Windows browser dispatch is out of scope. |
| `token` / [launch_browser_darwin](../s1/atoms/token/launch_browser_darwin.md) | Defer | macOS browser dispatch is out of scope. |
| `token` / [launch_browser_unix](../s1/atoms/token/launch_browser_unix.md) | Implement | Linux browser dispatch stays in scope. |
| `token` / [launch_browser_other](../s1/atoms/token/launch_browser_other.md) | Defer | Non-Linux fallback dispatch is out of scope for RR. |
| `quic` / [param_unix](../s1/atoms/quic/param_unix.md) | Implement | Linux QUIC socket tuning stays in scope. |
| `quic` / [param_windows](../s1/atoms/quic/param_windows.md) | Defer | Windows QUIC socket tuning is out of scope. |
| `fips` / [fips](../s1/atoms/fips/fips.md) | Defer | FIPS feature gating is explicitly excluded. |
| `fips` / [nofips](../s1/atoms/fips/nofips.md) | Implement | Non-FIPS baseline remains in scope. |

#### Runtime Detection

| Module / atom | RR decision | Scope note |
| --- | --- | --- |
| `cmd` / [linux_service](../s1/atoms/cmd/cloudflared/linux_service.md) | Implement systemd branch only | Keep runtime detection boundary, but Sysv behavior is FC-deferred. |
| `diagnostic` / [log_collector_host](../s1/atoms/diagnostic/log_collector_host.md) | Implement systemd host path only | Host log collection stays in scope, but Sysv-specific behavior is FC-deferred. |
| `diagnostic` / [log_collector](../s1/atoms/diagnostic/log_collector.md) | Implement | Docker, Kubernetes, and bare-host runtime selection stays in scope on Linux. |

## Feature MoSCoW

The tables below classify the 127 unique atoms extracted from the
[Features catalog](../s1/catalogs/cross-cutting/features/README.md). Each
atom is classified once under its primary contract owner. When a single atom
spans both in-scope and out-of-scope sub-behavior, the rationale column fixes
the RR subset explicitly.

### Derivation Method

Won't anchors are carried forward from
[dependency-decisions](dependency-decisions.md) and then cross-checked against
S1 behavioral evidence. All remaining feature atoms follow the confirmed RR
criteria below:

- Must: required for a QUIC tunnel to register, proxy a request, and shut down
  cleanly on Linux/systemd.
- Should: present in normal Go operation and relevant for parity, but not
  required for the minimal RR binary purpose.
- Could: rare, adjunct, or quality-of-life behavior intentionally sequenced
  behind the tunnel core.
- Won't: deferred beyond S4 or explicitly excluded from [cfdrs](../../../README.md).

The Access drop-in replacement rationale is formalized in
[ADR-017](adr/017-access-s4-drop-in-contract.md).

### Summary Counts

| Tier | Count | Meaning in RR |
| --- | ---: | --- |
| Must | 33 | Required for the Linux/systemd QUIC tunnel core to exist and shut down cleanly. |
| Should | 68 | Normal Go behavior expected to matter for parity, but not required for the minimal RR purpose. |
| Could | 18 | Rare, convenience, or adjunct behavior intentionally sequenced behind the core. |
| Won't | 8 | Explicitly outside RR/S4 scope. |
| Total | 127 | Unique stakeholder-scoped atoms classified exactly once. |

### Stakeholder 1 - Operator / Human

| Atom | Tier | Rationale |
| --- | --- | --- |
| [cmd/cloudflared/access/carrier](../s1/atoms/cmd/cloudflared/access/carrier.md) | Should | Access sidecar transport is normal authenticated-origin behavior. Reclassified from Could: drop-in replacement contract requires `cloudflared access` commands to be present at S4. |
| [cmd/cloudflared/access/cmd](../s1/atoms/cmd/cloudflared/access/cmd.md) | Should | Access CLI is normal authenticated-origin operator behavior. Reclassified from Could: drop-in replacement contract requires `cloudflared access` commands to be present at S4. |
| [cmd/cloudflared/access/validation](../s1/atoms/cmd/cloudflared/access/validation.md) | Should | Access input validation is normal authenticated-origin behavior. Reclassified from Could: drop-in replacement contract requires `cloudflared access` commands to be present at S4. |
| [cmd/cloudflared/cliutil/build_info](../s1/atoms/cmd/cloudflared/cliutil/build_info.md) | Should | Build info is normal operator-facing behavior but not required for tunnel traffic. |
| [cmd/cloudflared/flags/flags](../s1/atoms/cmd/cloudflared/flags/flags.md) | Must | Flag parsing is required to configure the RR binary. |
| [cmd/cloudflared/tail/cmd](../s1/atoms/cmd/cloudflared/tail/cmd.md) | Could | Remote log tailing is useful but not required for the tunnel core. |
| [cmd/cloudflared/tunnel/cmd](../s1/atoms/cmd/cloudflared/tunnel/cmd.md) | Must | Primary tunnel entrypoint is required for RR to run a tunnel at all. |
| [cmd/cloudflared/tunnel/configuration](../s1/atoms/cmd/cloudflared/tunnel/configuration.md) | Must | Tunnel CLI-to-config wiring is required for RR startup. |
| [cmd/cloudflared/tunnel/credential_finder](../s1/atoms/cmd/cloudflared/tunnel/credential_finder.md) | Must | Tunnel credentials are required for edge registration. |
| [cmd/cloudflared/tunnel/login](../s1/atoms/cmd/cloudflared/tunnel/login.md) | Could | Access login workflow is outside the RR tunnel core. |
| [cmd/cloudflared/tunnel/quick_tunnel](../s1/atoms/cmd/cloudflared/tunnel/quick_tunnel.md) | Should | Quick tunnels are common operator behavior but not required for the named-tunnel core. |
| [cmd/cloudflared/tunnel/signal](../s1/atoms/cmd/cloudflared/tunnel/signal.md) | Should | Reconnect signaling is normal operator control but not required for the first successful proxy path. |
| [cmd/cloudflared/tunnel/subcommand_context](../s1/atoms/cmd/cloudflared/tunnel/subcommand_context.md) | Should | Shared tunnel subcommand context is normal CLI behavior for named tunnels. |
| [cmd/cloudflared/tunnel/subcommand_context_teamnet](../s1/atoms/cmd/cloudflared/tunnel/subcommand_context_teamnet.md) | Could | Teamnet route flows are adjunct to the RR tunnel core. |
| [cmd/cloudflared/tunnel/subcommand_context_vnets](../s1/atoms/cmd/cloudflared/tunnel/subcommand_context_vnets.md) | Could | Virtual-network admin flows are adjunct to the RR tunnel core. |
| [cmd/cloudflared/tunnel/subcommands](../s1/atoms/cmd/cloudflared/tunnel/subcommands.md) | Should | Named-tunnel CLI workflow is normal operator behavior. |
| [cmd/cloudflared/tunnel/teamnet_subcommands](../s1/atoms/cmd/cloudflared/tunnel/teamnet_subcommands.md) | Could | IP route administration is useful but not required for the RR tunnel core. |
| [cmd/cloudflared/tunnel/vnets_subcommands](../s1/atoms/cmd/cloudflared/tunnel/vnets_subcommands.md) | Could | Virtual-network administration is useful but not required for the RR tunnel core. |
| [config/configuration](../s1/atoms/config/configuration.md) | Must | Config discovery and merge rules are required for RR startup. |
| [config/model](../s1/atoms/config/model.md) | Must | Config structs define the RR runtime contract. |
| [credentials/credentials](../s1/atoms/credentials/credentials.md) | Must | Origin cert and tunnel credential discovery are required for RR startup. |

### Stakeholder 8 - OS / Platform

| Atom | Tier | Rationale |
| --- | --- | --- |
| [cmd/cloudflared/app_forward_service](../s1/atoms/cmd/cloudflared/app_forward_service.md) | Should | Linux service/foreground composition is normal RR deployment behavior. |
| [cmd/cloudflared/app_service](../s1/atoms/cmd/cloudflared/app_service.md) | Should | Linux service composition is normal RR deployment behavior. |
| [cmd/cloudflared/common_service](../s1/atoms/cmd/cloudflared/common_service.md) | Should | Shared Linux service plumbing is normal deployment behavior. |
| [cmd/cloudflared/generic_service](../s1/atoms/cmd/cloudflared/generic_service.md) | Could | Generic fallback service behavior is not needed for the Tier 1 platform, but keeping the abstraction is low risk. |
| [cmd/cloudflared/linux_service](../s1/atoms/cmd/cloudflared/linux_service.md) | Should | Linux service installation remains in scope, but only the systemd branch. |
| [cmd/cloudflared/macos_service](../s1/atoms/cmd/cloudflared/macos_service.md) | Won't | launchd support is outside the RR platform boundary. |
| [cmd/cloudflared/service_template](../s1/atoms/cmd/cloudflared/service_template.md) | Should | Shared service templates remain relevant for the Linux systemd branch. |
| [cmd/cloudflared/updater/check](../s1/atoms/cmd/cloudflared/updater/check.md) | Won't | Automatic update is explicitly excluded from RR scope. |
| [cmd/cloudflared/updater/service](../s1/atoms/cmd/cloudflared/updater/service.md) | Won't | Update-as-service is explicitly excluded from RR scope. |
| [cmd/cloudflared/updater/update](../s1/atoms/cmd/cloudflared/updater/update.md) | Won't | Binary self-replacement is explicitly excluded from RR scope. |
| [cmd/cloudflared/updater/workers_service](../s1/atoms/cmd/cloudflared/updater/workers_service.md) | Won't | Workers update path is explicitly excluded from RR scope. |
| [cmd/cloudflared/updater/workers_update](../s1/atoms/cmd/cloudflared/updater/workers_update.md) | Won't | Workers update path is explicitly excluded from RR scope. |
| [cmd/cloudflared/windows_service](../s1/atoms/cmd/cloudflared/windows_service.md) | Won't | Windows SCM support is outside the RR platform boundary. |
| [fips/fips](../s1/atoms/fips/fips.md) | Won't | FIPS feature gating is explicitly excluded from RR scope. |
| [fips/nofips](../s1/atoms/fips/nofips.md) | Should | The non-FIPS baseline remains the RR default. |
| [overwatch/app_manager](../s1/atoms/overwatch/app_manager.md) | Must | Top-level app lifecycle management is required for controlled start and stop. |
| [overwatch/manager](../s1/atoms/overwatch/manager.md) | Must | Overwatch remains the top-level lifecycle coordinator. |
| [signal/safe_signal](../s1/atoms/signal/safe_signal.md) | Must | Graceful shutdown signaling is required for clean RR termination. |

### Stakeholder 2 - Cloudflare Edge

| Atom | Tier | Rationale |
| --- | --- | --- |
| [connection/control](../s1/atoms/connection/control.md) | Must | The control stream is required for edge registration and configuration exchange. |
| [connection/event](../s1/atoms/connection/event.md) | Should | Connection event broadcasting is normal edge-facing behavior but not strictly required for first proxy success. |
| [connection/observer](../s1/atoms/connection/observer.md) | Should | Observer sinks are normal runtime behavior and support health and quick-tunnel URL reporting. |
| [connection/protocol](../s1/atoms/connection/protocol.md) | Must | Protocol selection is required, but RR scopes this atom to the QUIC branch only. |
| [edgediscovery/protocol](../s1/atoms/edgediscovery/protocol.md) | Should | Protocol preference plumbing remains relevant, but the HTTP/2 fallback branch is deferred. |
| [features/features](../s1/atoms/features/features.md) | Should | Feature-list construction is exercised in normal edge registration. |
| [features/selector](../s1/atoms/features/selector.md) | Should | DNS-driven feature selection is normal edge negotiation behavior. |
| [tunnelrpc/pogs/registration_server](../s1/atoms/tunnelrpc/pogs/registration_server.md) | Must | Registration payload encoding is required for the edge handshake. |
| [tunnelrpc/registration_client](../s1/atoms/tunnelrpc/registration_client.md) | Must | Registration RPC client is required for the edge handshake. |
| [tunnelrpc/registration_server](../s1/atoms/tunnelrpc/registration_server.md) | Must | Registration server contract remains part of the RR handshake boundary. |

### Stakeholder 4 - Cloudflare Access

| Atom | Tier | Rationale |
| --- | --- | --- |
| [carrier/carrier](../s1/atoms/carrier/carrier.md) | Should | Access carrier transport is normal authenticated-origin behavior. Reclassified from Could: drop-in replacement contract requires `cloudflared access` commands to be present at S4. |
| [carrier/websocket](../s1/atoms/carrier/websocket.md) | Should | Access WebSocket carrier transport is normal authenticated-origin behavior. Reclassified from Could: drop-in replacement contract requires `cloudflared access` commands to be present at S4. |
| [credentials/origin_cert](../s1/atoms/credentials/origin_cert.md) | Should | Access origin-cert login flow is normal authenticated-origin behavior. Reclassified from Could: drop-in replacement contract requires `cloudflared access` commands to be present at S4. |
| [ingress/middleware/jwtvalidator](../s1/atoms/ingress/middleware/jwtvalidator.md) | Should | Access JWT validation is normal protected-origin behavior. |
| [ingress/middleware/middleware](../s1/atoms/ingress/middleware/middleware.md) | Should | Middleware composition is normal protected-origin behavior. |
| [management/middleware](../s1/atoms/management/middleware.md) | Should | Management middleware is normal authenticated dashboard behavior. |
| [management/token](../s1/atoms/management/token.md) | Should | Scoped token validation is normal authenticated dashboard behavior. |
| [token/encrypt](../s1/atoms/token/encrypt.md) | Should | Local token encryption is normal Access credential behavior. Reclassified from Could: drop-in replacement contract requires `cloudflared access` commands to be present at S4. |
| [token/path](../s1/atoms/token/path.md) | Should | Local token path handling is normal Access credential behavior. Reclassified from Could: drop-in replacement contract requires `cloudflared access` commands to be present at S4. |
| [token/token](../s1/atoms/token/token.md) | Should | Browser-driven Access login is normal authenticated-origin behavior. Reclassified from Could: drop-in replacement contract requires `cloudflared access` commands to be present at S4. |
| [token/transfer](../s1/atoms/token/transfer.md) | Should | Token callback transfer is normal Access credential behavior. Reclassified from Could: drop-in replacement contract requires `cloudflared access` commands to be present at S4. |
| [validation/validation](../s1/atoms/validation/validation.md) | Should | Access input validation is normal authenticated-origin behavior. Reclassified from Could: drop-in replacement contract requires `cloudflared access` commands to be present at S4. |

### Stakeholder 9 - Peer Connections (HA)

| Atom | Tier | Rationale |
| --- | --- | --- |
| [connection/tunnelsforha](../s1/atoms/connection/tunnelsforha.md) | Should | Multi-connection coordination is normal Go behavior but not strictly required for one successful RR connection. |
| [retry/backoffhandler](../s1/atoms/retry/backoffhandler.md) | Should | Controlled retry is normal runtime behavior, though not required for the first successful request path. |
| [supervisor/conn_aware_logger](../s1/atoms/supervisor/conn_aware_logger.md) | Could | Connection-aware log enrichment is useful but not required for the tunnel core. |
| [supervisor/external_control](../s1/atoms/supervisor/external_control.md) | Could | External reconnect triggers are adjunct operator behavior. |
| [supervisor/fuse](../s1/atoms/supervisor/fuse.md) | Should | First-connection gating is normal HA startup behavior. |
| [supervisor/metrics](../s1/atoms/supervisor/metrics.md) | Should | Supervisor metrics are normal operational behavior. |
| [supervisor/pqtunnels](../s1/atoms/supervisor/pqtunnels.md) | Should | PQ tunnel-state coordination is normal runtime behavior even though RR remains non-FIPS. |
| [supervisor/supervisor](../s1/atoms/supervisor/supervisor.md) | Must | The supervisor loop is required to own connection lifecycle. |
| [supervisor/tunnel](../s1/atoms/supervisor/tunnel.md) | Must | Per-connection tunnel lifecycle is required for RR to connect and reconnect cleanly. |
| [supervisor/tunnelsforha](../s1/atoms/supervisor/tunnelsforha.md) | Should | HA connection fan-out is normal Go behavior but not required for one successful RR connection. |
| [tunnelstate/conntracker](../s1/atoms/tunnelstate/conntracker.md) | Should | Active-connection tracking is normal readiness and diagnostics behavior. |

### Stakeholder 3 - Cloudflare Dashboard And API

| Atom | Tier | Rationale |
| --- | --- | --- |
| [cfapi/base_client](../s1/atoms/cfapi/base_client.md) | Should | Base REST plumbing is normal control-plane behavior for named tunnel management. |
| [cfapi/client](../s1/atoms/cfapi/client.md) | Should | Main REST client is normal control-plane behavior for named tunnel management. |
| [cfapi/hostname](../s1/atoms/cfapi/hostname.md) | Could | DNS hostname route administration is useful but not required for the RR tunnel core. |
| [cfapi/ip_route](../s1/atoms/cfapi/ip_route.md) | Could | Private IP route administration is useful but not required for the RR tunnel core. |
| [cfapi/ip_route_filter](../s1/atoms/cfapi/ip_route_filter.md) | Could | Private IP route filtering is adjunct control-plane behavior. |
| [cfapi/tunnel](../s1/atoms/cfapi/tunnel.md) | Should | Named-tunnel CRUD is normal RR operational behavior. |
| [cfapi/tunnel_filter](../s1/atoms/cfapi/tunnel_filter.md) | Should | Named-tunnel list/filter workflow is normal RR operational behavior. |
| [cfapi/virtual_network](../s1/atoms/cfapi/virtual_network.md) | Could | Virtual-network administration is adjunct control-plane behavior. |
| [cfapi/virtual_network_filter](../s1/atoms/cfapi/virtual_network_filter.md) | Could | Virtual-network filtering is adjunct control-plane behavior. |
| [management/events](../s1/atoms/management/events.md) | Should | Management event framing is normal dashboard behavior. |
| [management/logger](../s1/atoms/management/logger.md) | Should | Management log streaming is normal dashboard behavior. |
| [management/service](../s1/atoms/management/service.md) | Must | Management service is a critical-path RR atom and remains in scope. |
| [management/session](../s1/atoms/management/session.md) | Should | Session limiting and lifecycle are normal management behavior. |
| [orchestration/config](../s1/atoms/orchestration/config.md) | Should | Config update application is normal remotely-managed behavior. |
| [orchestration/orchestrator](../s1/atoms/orchestration/orchestrator.md) | Should | Remote config orchestration is normal remotely-managed behavior. |
| [tunnelrpc/pogs/configuration_manager](../s1/atoms/tunnelrpc/pogs/configuration_manager.md) | Should | Config export contract is normal edge/dashboard behavior. |
| [watcher/file](../s1/atoms/watcher/file.md) | Should | Local config watch remains normal management behavior. |

### Stakeholder 5 - Monitoring Systems

| Atom | Tier | Rationale |
| --- | --- | --- |
| [connection/metrics](../s1/atoms/connection/metrics.md) | Should | Connection metrics are normal operational behavior. |
| [datagramsession/metrics](../s1/atoms/datagramsession/metrics.md) | Should | Datagram metrics are normal operational behavior even though UDP is not the RR core focus. |
| [diagnostic/client](../s1/atoms/diagnostic/client.md) | Should | Remote diagnostic querying is normal operational behavior. |
| [diagnostic/diagnostic](../s1/atoms/diagnostic/diagnostic.md) | Should | Diagnostic model and packaging remain normal operational behavior. |
| [diagnostic/handlers](../s1/atoms/diagnostic/handlers.md) | Should | Diagnostic HTTP handlers remain normal operational behavior. |
| [flow/metrics](../s1/atoms/flow/metrics.md) | Could | Flow limiter metrics are useful but not required for the RR tunnel core. |
| [metrics/config](../s1/atoms/metrics/config.md) | Should | Metrics endpoint configuration remains normal operational behavior. |
| [metrics/metrics](../s1/atoms/metrics/metrics.md) | Should | Prometheus listener setup remains normal operational behavior. |
| [metrics/readiness](../s1/atoms/metrics/readiness.md) | Should | Readiness endpoint remains normal operational behavior. |
| [orchestration/metrics](../s1/atoms/orchestration/metrics.md) | Should | Remote-config metrics remain normal operational behavior. |
| [proxy/metrics](../s1/atoms/proxy/metrics.md) | Should | Proxy metrics remain normal operational behavior. |
| [quic/v3/metrics](../s1/atoms/quic/v3/metrics.md) | Could | Datagram-v3 metrics are useful but not required for the RR tunnel core. |
| [tunnelrpc/metrics/metrics](../s1/atoms/tunnelrpc/metrics/metrics.md) | Should | RPC metrics remain normal operational behavior. |

### Stakeholder 6 - Origin Services

| Atom | Tier | Rationale |
| --- | --- | --- |
| [connection/connection](../s1/atoms/connection/connection.md) | Should | Raw stream proxying is normal origin behavior, though HTTP proxy is the minimum RR path. |
| [hello/hello](../s1/atoms/hello/hello.md) | Could | Hello-world origin is useful for validation but not required for the RR tunnel core. |
| [ingress/config](../s1/atoms/ingress/config.md) | Must | Origin request config is required to proxy traffic correctly. |
| [ingress/ingress](../s1/atoms/ingress/ingress.md) | Must | Ingress matching is required to route traffic to origins. |
| [ingress/origin_connection](../s1/atoms/ingress/origin_connection.md) | Must | Origin connection management is required to proxy traffic correctly. |
| [ingress/origin_dialer](../s1/atoms/ingress/origin_dialer.md) | Must | Origin dialing is required to proxy traffic correctly. |
| [ingress/origin_icmp_proxy](../s1/atoms/ingress/origin_icmp_proxy.md) | Should | ICMP proxying is normal Go behavior but not required for the RR tunnel core. |
| [ingress/origin_proxy](../s1/atoms/ingress/origin_proxy.md) | Must | Origin proxy orchestration is required to proxy traffic correctly. |
| [ingress/origin_service](../s1/atoms/ingress/origin_service.md) | Must | Origin service selection is required, but RR excludes the bastion / SSH-server subset. |
| [ingress/packet_router](../s1/atoms/ingress/packet_router.md) | Should | Packet routing is normal Go behavior but not required for the RR tunnel core. |
| [ingress/rule](../s1/atoms/ingress/rule.md) | Must | Rule semantics are required to route traffic correctly. |
| [ipaccess/access](../s1/atoms/ipaccess/access.md) | Could | Source-IP policy enforcement is useful but not required for the RR tunnel core. |
| [proxy/proxy](../s1/atoms/proxy/proxy.md) | Must | HTTP proxy behavior is required for RR to serve requests. |
| [socks/connection_handler](../s1/atoms/socks/connection_handler.md) | Should | Ingress-side SOCKS5 proxying remains normal Go behavior and is distinct from the excluded outbound client surface. |
| [socks/request_handler](../s1/atoms/socks/request_handler.md) | Should | Ingress-side SOCKS5 request handling remains normal Go behavior. |
| [stream/stream](../s1/atoms/stream/stream.md) | Should | Stream relay behavior is normal origin behavior. |
| [websocket/connection](../s1/atoms/websocket/connection.md) | Should | WebSocket relay behavior is normal origin behavior. |
| [websocket/websocket](../s1/atoms/websocket/websocket.md) | Should | WebSocket proxying remains normal Go behavior. |

### Stakeholder 7 - DNS Infrastructure

| Atom | Tier | Rationale |
| --- | --- | --- |
| [edgediscovery/allregions/address](../s1/atoms/edgediscovery/allregions/address.md) | Must | Address selection is required to dial edge endpoints. |
| [edgediscovery/allregions/discovery](../s1/atoms/edgediscovery/allregions/discovery.md) | Must | SRV discovery is required to find edge endpoints. |
| [edgediscovery/allregions/region](../s1/atoms/edgediscovery/allregions/region.md) | Must | Region modeling is required to choose edge endpoints. |
| [edgediscovery/allregions/regions](../s1/atoms/edgediscovery/allregions/regions.md) | Must | Region pool management is required to choose edge endpoints. |
| [edgediscovery/allregions/usedby](../s1/atoms/edgediscovery/allregions/usedby.md) | Must | Address usage tracking is required to reuse and rotate edge endpoints safely. |
| [edgediscovery/dial](../s1/atoms/edgediscovery/dial.md) | Must | Edge dialing is required for RR to connect at all. |
| [edgediscovery/edgediscovery](../s1/atoms/edgediscovery/edgediscovery.md) | Must | Edge discovery orchestration is required for RR to connect at all. |

## Explicit Non-Ports

The list below records every explicit RR non-port while preserving the current
stakeholder MoSCoW classification. Deferred items remain valid S1 evidence for
later phases; excluded items are architectural decisions for [cfdrs](../../../README.md) itself.

### RR-Deferred (FC / Post-S4 Backlog)

| Feature | Target window | Evidence | Blocking reason |
| --- | --- | --- | --- |
| macOS platform support | FC | [platform-substrates](../s1/catalogs/cross-cutting/platform-substrates.md), [platforms](../s1/catalogs/domain/platforms.md), [runtime-pattern-friction](../s1/catalogs/cross-cutting/porting-friction/runtime-pattern-friction.md) | Requires additional `darwin`-gated service, browser, diagnostics, and ICMP branches. This is Rank 6 build-tag friction with no benefit to Tier 1 RR delivery. |
| Windows platform support | FC | [platform-substrates](../s1/catalogs/cross-cutting/platform-substrates.md), [platforms](../s1/catalogs/domain/platforms.md), [runtime-pattern-friction](../s1/catalogs/cross-cutting/porting-friction/runtime-pattern-friction.md) | Requires Windows SCM, QUIC parameter, browser, diagnostics, and ICMP branches. This is Rank 6 build-tag friction plus Windows-specific operational divergence. |
| FreeBSD platform support | FC | [platform-substrates](../s1/catalogs/cross-cutting/platform-substrates.md), [platforms](../s1/catalogs/domain/platforms.md) | Browser launch is partially covered by the Unix helper, but service management and ICMP lack a dedicated RR path. |
| Non-systemd init paths (`sysv`, launchd, SCM) | Post-S4 or FC | [platform-substrates](../s1/catalogs/cross-cutting/platform-substrates.md), [platforms](../s1/catalogs/domain/platforms.md) | RR keeps a single Linux service target to avoid multiplying operational paths before parity is proven. |
| Non-x86-64 architectures | FC | [platform-substrates](../s1/catalogs/cross-cutting/platform-substrates.md), [dependency-decisions](dependency-decisions.md) | RR keeps a single architecture target to reduce packaging, deployment, and benchmarking variance while S3 is still forming. |
| FIPS build mode | Post-S3 / S4 verification | [operator-and-platform](../s1/catalogs/cross-cutting/features/operator-and-platform.md), [platform-substrates](../s1/catalogs/cross-cutting/platform-substrates.md), [runtime-pattern-friction](../s1/catalogs/cross-cutting/porting-friction/runtime-pattern-friction.md) | The `fips` build tag swaps the TLS backend at compile time. That is Rank 6 build-tag friction with a high-blast-radius crypto backend split, so RR stays on the audited non-FIPS baseline. |
| HTTP/2 tunnel transport | Post-S3 | [edge-and-access](../s1/catalogs/cross-cutting/features/edge-and-access.md), [type-system-friction](../s1/catalogs/cross-cutting/porting-friction/type-system-friction.md), [runtime-pattern-friction](../s1/catalogs/cross-cutting/porting-friction/runtime-pattern-friction.md) | QUIC is sufficient for RR to register and proxy traffic. The HTTP/2 path adds extra `io.ReadWriteCloser`, `defer`, and panic-boundary translation work without changing the Tier 1 parity target. |
| QUIC 0-RTT | Post-S4 benchmarks | [edge-and-access](../s1/catalogs/cross-cutting/features/edge-and-access.md), [dependency-decisions](dependency-decisions.md) | `registerConnection` is unsafe for early data and 0-RTT is not required for RR parity. |
| `io_uring` runtime | Post-S4 benchmarks | [dependency-decisions](dependency-decisions.md) | S2.1 already marked `io_uring` out because it raises the kernel floor and runtime complexity before parity is proven. |
| `secrecy` + `zeroize` boundary hardening | Post-S3 crypto boundary | [dependency-decisions](dependency-decisions.md) | S2.1 explicitly deferred this to the crypto crate boundary rather than the initial RR runtime cut. |
| SSH Server / bastion origin service | FC after upstream completion | [api-monitoring-origin-dns](../s1/catalogs/cross-cutting/features/api-monitoring-origin-dns.md), [type-system-friction](../s1/catalogs/cross-cutting/porting-friction/type-system-friction.md), [cmd/cloudflared/tunnel/configuration](../s1/atoms/cmd/cloudflared/tunnel/configuration.md) | The bastion subset expands the already-large `OriginService` variant set, and upstream still marks the SSH-server flag hidden/incomplete. |
| XDP / AF_XDP | Roadmap post-FC | [dependency-decisions](dependency-decisions.md) | Kernel-bypass networking is not part of the RR parity surface and remains an architecture-level future optimization. |
| NUMA-aware allocation | FC if deployment target changes | [dependency-decisions](dependency-decisions.md) | RR assumes a single-socket target and does not need NUMA-aware allocation behavior in S3 or S4. |

### Permanently Excluded

| Feature | Evidence | Justification |
| --- | --- | --- |
| SOCKS5 outbound client | [operator-and-platform](../s1/catalogs/cross-cutting/features/operator-and-platform.md), [api-monitoring-origin-dns](../s1/catalogs/cross-cutting/features/api-monitoring-origin-dns.md), [config/model](../s1/atoms/config/model.md), [cmd/cloudflared/access/carrier](../s1/atoms/cmd/cloudflared/access/carrier.md) | Deliberate Go omission. `access tcp` exposes a plain TCP listener, `Forwarder` has no SOCKS5 client fields, and the only upstream SOCKS5 surface is the ingress-side inbound proxy. |
| Automatic update | [operator-and-platform](../s1/catalogs/cross-cutting/features/operator-and-platform.md), [platform-substrates](../s1/catalogs/cross-cutting/platform-substrates.md) | Permanently excluded by [cfdrs](../../../README.md) design. The binary is expected to be managed by systemd and packaging workflows, not by self-replacement inside a security-sensitive daemon. |
| `update` command | [operator-and-platform](../s1/catalogs/cross-cutting/features/operator-and-platform.md) | Depends on the auto-updater surface, which is permanently excluded from [cfdrs](../../../README.md). |
| [glommio](https://crates.io/crates/glommio) runtime | [dependency-decisions](dependency-decisions.md) | Permanently excluded in S2.1 due to QUIC transport incompatibility and ecosystem lockout. |
| Foundations crate as a direct dependency | [dependency-decisions](dependency-decisions.md) | Out as a direct dependency. It may appear transitively, but [cfdrs](../../../README.md) does not depend on the full Cloudflare bootstrap facade directly. |
| pprof handlers with Go semantics | [metrics/metrics](../s1/atoms/metrics/metrics.md), [management/service](../s1/atoms/management/service.md), [dependency-decisions](dependency-decisions.md) | Go `net/http/pprof` does not have a direct Rust equivalent with matching semantics. RR treats this as replaceable debug tooling rather than parity-critical behavior. |

## S2.2 Exit Gate

| Criterion | Status |
| --- | --- |
| Platform matrix stays coherent with Layer 9 of [dependency-decisions](dependency-decisions.md) | Yes |
| macOS, Windows, and FreeBSD exclusions are explicitly documented | Yes |
| Stage 3.9 impact is recorded for later S2.8 planning | Yes |
| Every Must atom remains traceable to S1 evidence or S2.1 decisions | Yes |
| Every Should atom remains traceable to S1 evidence | Yes |
| Won't and non-port entries stay anchored to S1 findings or S2.1 decisions | Yes |
| Two non-port buckets are complete: deferred and permanently excluded | Yes |
| SOCKS5 outbound exclusion is justified with explicit Go evidence | Yes |
| Auto-update and `update` command exclusions are justified as [cfdrs](../../../README.md) design decisions | Yes |

## Notes

- The seven critical-path atoms from [audit-analysis](../s1/audit-analysis.md)
  all remain in `Must`: [connection/control](../s1/atoms/connection/control.md),
  [connection/protocol](../s1/atoms/connection/protocol.md),
  [management/service](../s1/atoms/management/service.md),
  [quic/v3/session](../s1/atoms/quic/v3/session.md),
  [supervisor/tunnel](../s1/atoms/supervisor/tunnel.md),
  [tunnel/configuration](../s1/atoms/cmd/cloudflared/tunnel/configuration.md),
  and [tunnel/cmd](../s1/atoms/cmd/cloudflared/tunnel/cmd.md). RR does not
  currently classify [quic/v3/session](../s1/atoms/quic/v3/session.md) in the
  stakeholder feature set because it is not one of the 127 stakeholder atoms;
  it stays mandatory at the architecture level.
- `Should` remains a parity obligation for S4 even when the feature is not on
  the first S3 critical path.
- `Could` is not an abandonment bucket. It means the behavior exists in the
  Go baseline but is intentionally sequenced after the RR tunnel core.
