# Platform-Specific Features and Behaviors

- Baseline date: 20260324
- Baseline reference: [cloudflare/cloudflared/tree/2026.3.0](https://github.com/cloudflare/cloudflared/tree/2026.3.0)

This document catalogs every cloudflared feature or behavior that varies by
operating system, container runtime, init system, or build-time feature flag.
Each entry links to the baseline audit atom that describes the Go
implementation in detail.

Atom links are relative to [atoms/](../../atoms/).

---

## Substrate Dimensions

| Dimension | Values | Selection Mechanism |
|-----------|--------|---------------------|
| **Operating System** | Linux, macOS (darwin), Windows, FreeBSD, generic fallback | Go build tags / file suffix |
| **Init System** | Systemd, Sysvinit, Launchd, Windows SCM | Runtime detection (Linux); compile-time (others) |
| **Container Runtime** | Docker, Kubernetes, bare host | Runtime detection |
| **Build Feature** | FIPS, non-FIPS | Go build tag (`fips` / `!fips`) |
| **Socket Family** | Unix-domain, Windows named pipe | Build tag (`unix` / `windows`) |
| **Architecture** | amd64, arm64 | Build-time (`GOARCH`); no arch-gated source files but separate Docker images, MSI installers, and package artifacts |

---

## Platform Support Matrix

Features marked with their behavior on each platform.
**Full** = complete implementation, **Partial** = limited or degraded,
**Stub** = compiles but returns an error, **N/A** = not applicable.

### ICMP Proxy

| Capability | Linux | macOS | Windows | POSIX Generic | Other |
|------------|:-----:|:-----:|:-------:|:-------------:|:-----:|
| Socket type | `SOCK_DGRAM` | Raw socket | FFI (`iphlpapi.dll`) | `x/net/icmp` | Stub |
| Unprivileged ICMP | Full | N/A (needs root) | N/A (admin) | Partial | Stub |
| IPv4 echo | Full | Full | Full | Full | Stub |
| IPv6 echo | Full | Full | Full | Full | Stub |
| Ping-group detection | Full | N/A | N/A | N/A | N/A |
| `/proc/net/route` gateway | Full | N/A | N/A | N/A | N/A |
| Echo-ID tracking (`sync.Map`) | N/A | Full | N/A | Full | N/A |
| Unsafe pointer arithmetic | N/A | N/A | Full | N/A | N/A |

Source atoms:

- Linux — [ingress/icmp_linux.md](../../atoms/ingress/icmp_linux.md)
- macOS — [ingress/icmp_darwin.md](../../atoms/ingress/icmp_darwin.md)
- Windows — [ingress/icmp_windows.md](../../atoms/ingress/icmp_windows.md)
- POSIX — [ingress/icmp_posix.md](../../atoms/ingress/icmp_posix.md)
- Fallback — [ingress/icmp_generic.md](../../atoms/ingress/icmp_generic.md)

### Service Management

| Capability | Linux (Systemd) | Linux (Sysvinit) | macOS (Launchd) | Windows (SCM) |
|------------|:---------------:|:-----------------:|:---------------:|:-------------:|
| Install service | Full | Full | Full | Full |
| Uninstall service | Full | Full | Full | Full |
| Service readiness signal | `Type=notify` | N/A | N/A | `svc.Status` |
| Recovery policy | `Restart=on-failure` | N/A | `KeepAlive` plist key | `configRecoveryOption()` |
| Log routing | journald | syslog / file | stdout/stderr redirect | Windows Event Log |
| Root/admin detection | N/A | N/A | `isRootUser()` | Implicit (SCM requires admin) |
| Init-system detection | Runtime (`/run/systemd/system`) | Fallback when systemd absent | Compile-time | Compile-time |
| Unit/config format | `.service` INI template | Shell script (`/etc/init.d/`) | `.plist` XML | Win32 API calls |

Source atoms:

- Linux — [cmd/cloudflared/linux_service.md](../../atoms/cmd/cloudflared/linux_service.md)
- macOS — [cmd/cloudflared/macos_service.md](../../atoms/cmd/cloudflared/macos_service.md)
- Windows — [cmd/cloudflared/windows_service.md](../../atoms/cmd/cloudflared/windows_service.md)
- Dispatcher — [cmd/cloudflared/generic_service.md](../../atoms/cmd/cloudflared/generic_service.md)
- Template — [cmd/cloudflared/service_template.md](../../atoms/cmd/cloudflared/service_template.md)

### System Diagnostics

| Data Point | Linux | macOS | Windows |
|------------|:-----:|:-----:|:-------:|
| Memory info | `/proc/meminfo` | `sysctl hw.memsize` | WMI `Win32_ComputerSystem` |
| CPU info | `/proc/cpuinfo` or `uname` | `sysctl hw.physicalcpu` | WMI |
| Disk info | N/A | N/A | WMI `Win32_LogicalDisk` |
| File descriptors | `/proc/fd/` count | `sysctl` | N/A |
| OS version | `lsb_release` subprocess | `sysctl` | WMI `Win32_OperatingSystem` |
| Kernel version | `uname -r` | `sysctl kern.osrelease` | WMI |

Source atoms:

- Linux — [diagnostic/system_collector_linux.md](../../atoms/diagnostic/system_collector_linux.md)
- macOS — [diagnostic/system_collector_macos.md](../../atoms/diagnostic/system_collector_macos.md)
- Windows — [diagnostic/system_collector_windows.md](../../atoms/diagnostic/system_collector_windows.md)
- Shared — [diagnostic/system_collector_utils.md](../../atoms/diagnostic/system_collector_utils.md)

### Network Path Tracing

| Aspect | Unix (Linux/macOS/BSD) | Windows |
|--------|:----------------------:|:-------:|
| Command | `traceroute` | `tracert` |
| Output parsing | Hop + RTT regex | Windows-format regex |
| Build tag | `//go:build unix` | `//go:build windows` |

Source atoms:

- Unix — [diagnostic/network/collector_unix.md](../../atoms/diagnostic/network/collector_unix.md)
- Windows — [diagnostic/network/collector_windows.md](../../atoms/diagnostic/network/collector_windows.md)

### Browser Launching (Auth Flow)

| Platform | Command | Build Tag | Notes |
|----------|---------|-----------|-------|
| Windows | `cmd /c start "<URL>"` | `windows` | `CREATE_NO_WINDOW` via `SysProcAttr` |
| macOS | `open "<URL>"` | `darwin` | Native default-app handler |
| Linux / FreeBSD | `xdg-open "<URL>"` | `linux \|\| freebsd` | Desktop environment standard |
| Other | Best-effort `open` | Catch-all | Fallback for unsupported platforms |

Common facade: all variants implement `getBrowserCmd(url) *exec.Cmd`.

Source atoms:

- Windows — [token/launch_browser_windows.md](../../atoms/token/launch_browser_windows.md)
- macOS — [token/launch_browser_darwin.md](../../atoms/token/launch_browser_darwin.md)
- Linux/FreeBSD — [token/launch_browser_unix.md](../../atoms/token/launch_browser_unix.md)
- Fallback — [token/launch_browser_other.md](../../atoms/token/launch_browser_other.md)

### QUIC Socket Parameters

| Parameter Set | Build Tag | Purpose |
|---------------|-----------|---------|
| Unix | `//go:build unix` | Unix-specific socket options and buffer sizes |
| Windows | `//go:build windows` | Windows-specific socket options and buffer sizes |

Source atoms:

- Unix — [quic/param_unix.md](../../atoms/quic/param_unix.md)
- Windows — [quic/param_windows.md](../../atoms/quic/param_windows.md)

### Auto-Updater

| Capability | Unix (Linux/macOS) | Windows |
|------------|:------------------:|:-------:|
| Auto-update supported | Full | Partial (batch workaround) |
| Binary replacement | Atomic rename + `exec()` | Batch script: write → spawn → exit → rename |
| Process restart | In-place exec-based takeover | New process via batch |
| Cleanup | None needed | Temporary batch file deletion |

Source atoms:

- Updater — [cmd/cloudflared/updater/update.md](../../atoms/cmd/cloudflared/updater/update.md)
- Workers update — [cmd/cloudflared/updater/workers_update.md](../../atoms/cmd/cloudflared/updater/workers_update.md)

### Configuration Discovery

| Platform | Default Search Paths |
|----------|---------------------|
| Linux / Unix | `~/.cloudflare-warp/config.yaml`, `/etc/cloudflare-warp/`, system paths |
| macOS | `~/Library/Application Support/Cloudflare/`, system admin paths |
| Windows | `%APPDATA%\Cloudflare\` |

Source atom: [config/configuration.md](../../atoms/config/configuration.md)

---

## Container Runtime Matrix

Log collection behavior depends on the detected container runtime,
not the operating system.

| Capability | Docker | Kubernetes | Bare Host |
|------------|:------:|:----------:|:---------:|
| Log source | `docker logs <id>` | `kubectl logs <pod>` | `journalctl` / log file |
| Detection | Runtime | Runtime | Runtime |
| Init-system awareness | N/A | N/A | Systemd ↔ Sysvinit dispatch |
| Subprocess | `docker` CLI | `kubectl` CLI | `journalctl` or file read |

Source atoms:

- Docker — [diagnostic/log_collector_docker.md](../../atoms/diagnostic/log_collector_docker.md)
- Kubernetes — [diagnostic/log_collector_kubernetes.md](../../atoms/diagnostic/log_collector_kubernetes.md)
- Host — [diagnostic/log_collector_host.md](../../atoms/diagnostic/log_collector_host.md)
- Shared — [diagnostic/log_collector_utils.md](../../atoms/diagnostic/log_collector_utils.md)
- Dispatch — [diagnostic/log_collector.md](../../atoms/diagnostic/log_collector.md)

---

## FIPS Build-Feature Matrix

FIPS is a **compile-time** feature flag, not a runtime toggle. A single
binary is either FIPS-enabled or not.

| Aspect | `fips` build | `!fips` build |
|--------|:------------:|:-------------:|
| TLS backend | `crypto/tls/fipsonly` | Standard Go `crypto/tls` |
| `IsFipsEnabled()` | `true` | `false` |
| Approved curves | P-256, P-384 only | All curves including X25519 |
| Post-quantum (strict) | `P256Kyber768Draft00` (0xfe32) | `X25519MLKEM768` (0x11ec) |
| Post-quantum (prefer) | PQ-first FIPS + classical P-256 | PQ-first + fallback curves |
| Post-quantum (off) | P-256 only | Standard negotiation |

Source atoms:

- FIPS — [fips/fips.md](../../atoms/fips/fips.md)
- Non-FIPS — [fips/nofips.md](../../atoms/fips/nofips.md)
- TLS config — [tlsconfig/tlsconfig.md](../../atoms/tlsconfig/tlsconfig.md)

---

## CPU Architecture

Unlike OS-level gating, cloudflared has **no `GOARCH`-conditional source files**.
All Go code compiles identically on amd64 and arm64. Architecture differences
surface at build, packaging, and distribution time — plus a handful of runtime
contracts that care about byte order or pointer width.

### Build and Distribution Matrix

| Artifact | amd64 | arm64 | 32-bit (x86) | Notes |
|----------|:-----:|:-----:|:------------:|-------|
| Docker image | Dedicated `Dockerfile.amd64` | Dedicated `Dockerfile.arm64` | N/A | Multi-arch manifest; base `gcr.io/distroless/base-debian13:nonroot` |
| Linux deb/rpm | Built per-arch via `fpm` | Built per-arch via `fpm` | Not shipped | Filename follows `<name>_<ver>_<arch>` convention |
| Windows MSI (64-bit) | `%ProgramFiles%\cloudflared\` | N/A | N/A | 64-bit install path |
| Windows MSI (32-bit) | N/A | N/A | `%ProgramFiles(x86)%\cloudflared\` | 32-bit install path |
| FIPS binary | `cloudflared-fips` | Likely supported (BoringSSL covers arm64) | N/A | BoringSSL arch constraints not explicitly documented |
| Standard binary | `cloudflared` | `cloudflared` | N/A | `CGO_ENABLED=0` static build |

Source: [build-packaging.md](../domain/deployments/build-packaging.md)

### Build-Time Compilation Flags

| Flag | Value | Purpose |
|------|-------|---------|
| `CGO_ENABLED` | `0` (standard) / `1` (FIPS) | Static build eliminates native C arch mismatches |
| `GOOS` | `linux`, `darwin`, `windows` | Target OS selection |
| `GOARCH` | `amd64`, `arm64` | Target CPU architecture |
| Linker vars | `main.Version`, `main.BuildTime`, `updater.BuiltForPackageManager`, `metrics.Runtime`, `main.BuildType` | Embedded at compile time; arch-agnostic |
| FIPS ldflags | `-linkmode=external -extldflags=-static` | Required for BoringSSL linkage; tags: `osusergo netgo fips` |

### Runtime Architecture Detection

The diagnostic subsystem captures the CPU architecture at startup for
reporting purposes.

| Collector Field | Source | Purpose |
|----------------|--------|---------|
| `architecture` | OS-reported (e.g., `uname -m`) | Hardware architecture string |
| `goArchitecture` | `runtime.GOARCH` | Go runtime architecture (may differ under emulation) |

Source atom: [diagnostic/system_collector.md](../../atoms/diagnostic/system_collector.md)

### Wire-Format Endianness Contracts

Although Go handles endianness transparently for most operations, several
wire protocols enforce explicit big-endian encoding that the Rust port
must preserve.

| Protocol | Endianness | Encoding Mechanism | Atoms |
|----------|:----------:|-------------------|-------|
| QUIC datagram v2/v3 framing | Big-endian | `encoding/binary.BigEndian` | [quic/v3/datagram.md](../../atoms/quic/v3/datagram.md) |
| RPC / capnp wire format | Big-endian | capnp schema-driven | [tunnelrpc/](../../atoms/tunnelrpc/) atoms |
| SOCKS5 port encoding | Big-endian | Manual 2-byte pack | [socks/](../../atoms/socks/) atoms |
| FNV hash for canary rollout | N/A (32-bit) | `hash/fnv.New32a()` | [connection/protocol.md](../../atoms/connection/protocol.md) |

Rust porting note: use `u16::to_be_bytes()` / `u32::to_be_bytes()` or the
`byteorder` crate to match Go's `binary.BigEndian` encoding.

### Auto-Updater Architecture Awareness

The updater queries `https://update.argotunnel.com` for new binaries.
The base URL is passed as an opaque `url string` parameter — the atoms
**do not document** how the endpoint resolves to an architecture-specific
binary. The likely mechanism is server-side routing based on client
version metadata or `runtime.GOARCH`, but this is a **documentation gap**
in the baseline audit.

Source atoms:

- [cmd/cloudflared/updater/update.md](../../atoms/cmd/cloudflared/updater/update.md)
- [cmd/cloudflared/updater/workers_update.md](../../atoms/cmd/cloudflared/updater/workers_update.md)
- [cmd/cloudflared/updater/workers_service.md](../../atoms/cmd/cloudflared/updater/workers_service.md)

---

## Gating Mechanism Inventory

Every platform-gated source unit, grouped by mechanism.

### Compile-Time: Go Build Tags / File Suffix

| Module | Atom | `linux` | `darwin` | `windows` | `unix` | `!windows` | `fips` | `!fips` | Fallback |
|--------|------|:-------:|:--------:|:---------:|:------:|:-----------:|:------:|:-------:|:--------:|
| ingress | [icmp_linux](../../atoms/ingress/icmp_linux.md) | X | | | | | | | |
| ingress | [icmp_darwin](../../atoms/ingress/icmp_darwin.md) | | X | | | | | | |
| ingress | [icmp_windows](../../atoms/ingress/icmp_windows.md) | | | X | | | | | |
| ingress | [icmp_posix](../../atoms/ingress/icmp_posix.md) | | | | | X | | | |
| ingress | [icmp_generic](../../atoms/ingress/icmp_generic.md) | | | | | | | | X |
| diagnostic | [sys_collector_linux](../../atoms/diagnostic/system_collector_linux.md) | X | | | | | | | |
| diagnostic | [sys_collector_macos](../../atoms/diagnostic/system_collector_macos.md) | | X | | | | | | |
| diagnostic | [sys_collector_windows](../../atoms/diagnostic/system_collector_windows.md) | | | X | | | | | |
| diagnostic | [collector_unix](../../atoms/diagnostic/network/collector_unix.md) | | | | X | | | | |
| diagnostic | [collector_windows](../../atoms/diagnostic/network/collector_windows.md) | | | X | | | | | |
| cmd | [macos_service](../../atoms/cmd/cloudflared/macos_service.md) | | X | | | | | | |
| cmd | [windows_service](../../atoms/cmd/cloudflared/windows_service.md) | | | X | | | | | |
| token | [browser_windows](../../atoms/token/launch_browser_windows.md) | | | X | | | | | |
| token | [browser_darwin](../../atoms/token/launch_browser_darwin.md) | | X | | | | | | |
| token | [browser_unix](../../atoms/token/launch_browser_unix.md) | X | | | | | | | |
| token | [browser_other](../../atoms/token/launch_browser_other.md) | | | | | | | | X |
| quic | [param_unix](../../atoms/quic/param_unix.md) | | | | X | | | | |
| quic | [param_windows](../../atoms/quic/param_windows.md) | | | X | | | | | |
| fips | [fips](../../atoms/fips/fips.md) | | | | | | X | | |
| fips | [nofips](../../atoms/fips/nofips.md) | | | | | | | X | |

### Runtime Detection

| Module | Atom | Detection Method | Detected Values |
|--------|------|-----------------|-----------------|
| cmd | [linux_service](../../atoms/cmd/cloudflared/linux_service.md) | Filesystem probe (`/run/systemd/system`) | Systemd, Sysvinit |
| diagnostic | [log_collector](../../atoms/diagnostic/log_collector.md) | Environment / socket probing | Docker, Kubernetes, bare host |
| diagnostic | [log_collector_host](../../atoms/diagnostic/log_collector_host.md) | Init-system check | Systemd ↔ Sysvinit log path |

---

## Architectural Patterns

### Pattern 1 — Complete Module Replacement

Each platform gets its own file implementing the same interface. Go's
file-suffix convention ensures exactly one file compiles per target.
Applies to: ICMP proxy, system diagnostic collectors, browser launcher,
QUIC parameters.

```text
ingress/
  icmp_linux.go      ← only on linux
  icmp_darwin.go     ← only on darwin
  icmp_windows.go    ← only on windows
  icmp_posix.go      ← only on !windows
  icmp_generic.go    ← unsupported fallback
```

### Pattern 2 — Fallback / Stub Implementations

Unsupported platforms compile a stub that always returns an error. This
keeps the interface satisfied at the type level while clearly signaling
"not available here" at runtime. Examples:
[icmp_generic](../../atoms/ingress/icmp_generic.md),
[browser_other](../../atoms/token/launch_browser_other.md).

### Pattern 3 — Runtime Init-System Detection (Linux Only)

The Linux service installer is **not** gated by build tags. Instead it
probes `/run/systemd/system` at runtime and selects either the Systemd
or Sysvinit code path. This lets a single binary work on both init
systems.

### Pattern 4 — Build-Time Feature Toggle (FIPS)

Two mutually exclusive files governed by a custom build tag (`fips` /
`!fips`). The FIPS variant imports `crypto/tls/fipsonly`, which
fundamentally changes the TLS backend at link time. There is no
runtime fallback.

### Pattern 5 — Container Runtime Detection

Log collectors detect the container runtime (Docker / Kubernetes / bare
host) entirely at runtime, with no build-tag involvement. The dispatch
logic lives in a shared orchestrator that probes for `docker`, `kubectl`,
or falls through to host-level log sources.

---

## Rust Porting Implications

| Go Mechanism | Rust Equivalent | Notes |
|-------------|-----------------|-------|
| `//go:build linux` | `#[cfg(target_os = "linux")]` | Direct mapping |
| `//go:build darwin` | `#[cfg(target_os = "macos")]` | Note: Go uses `darwin`, Rust uses `macos` |
| `//go:build windows` | `#[cfg(target_os = "windows")]` | Direct mapping |
| `//go:build unix` | `#[cfg(unix)]` | Covers Linux, macOS, FreeBSD |
| `//go:build !windows` | `#[cfg(not(target_os = "windows"))]` | Or `#[cfg(unix)]` when appropriate |
| `//go:build fips` | `#[cfg(feature = "fips")]` | Cargo feature flag |
| File suffix `_linux.go` | Separate module file + `#[cfg]` | No automatic suffix convention in Rust |
| `runtime.GOOS` | `std::env::consts::OS` | Runtime OS detection |
| `x509.SystemCertPool()` | `rustls-native-certs` crate | Platform cert store access |
| `syscall.NewLazyDLL` (Windows) | `windows-sys` crate FFI | Safe wrappers recommended |
| `sysctl` subprocess | `sysctl` crate (macOS) | Native API preferred over subprocess |
| `/proc` filesystem reads | `procfs` crate (Linux) | Typed wrappers over raw file reads |
| WMI queries (Windows) | `wmi` crate | COM-based queries |
| `runtime.GOARCH` | `std::env::consts::ARCH` | Runtime architecture string |
| `GOARCH` build var | `#[cfg(target_arch = "x86_64")]` / `#[cfg(target_arch = "aarch64")]` | Compile-time arch selection |
| `encoding/binary.BigEndian` | `u32::to_be_bytes()` / `byteorder` crate | Explicit endianness for wire formats |
