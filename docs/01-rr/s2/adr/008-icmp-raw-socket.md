# ADR-008 - ICMP Raw Socket

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Decided |
| Origin | Layer 9 in [dependency-decisions](../dependency-decisions.md) |
| Risk linkage | R2.6 in [risks](../risks.md) |

## Question

Should [cfdrs](../../../../README.md) implement ICMP proxying using a third-party library
or a custom implementation using [socket2](https://crates.io/crates/socket2) plus a dedicated unsafe boundary?

## Decision

Custom implementation using [socket2](https://crates.io/crates/socket2), with ICMP raw socket operations
isolated inside a dedicated unsafe crate. Same build-not-buy logic as
ADR-007 (SOCKS5). [nix](https://crates.io/crates/nix) remains in scope for signal
handling, and general socket options, but not for ICMP raw sockets.

## Rationale and Evidence

Linux ICMP path requires `SOCK_DGRAM` unprivileged sockets,
ping-group detection via `/proc/sys/net/ipv4/ping_group_range`, per-platform
echo-ID tracking, and integration with the datagram session packet router. No
library wraps this surface correctly for the [cfdrs](../../../../README.md) deployment model. The Go
implementation is self-contained in `ingress/icmp_linux.go` using raw OS
primitives. The implementation boundary in Rust is explicit: all unsafe code
(including ICMP raw socket handling) lives in a dedicated unsafe crate, while
safe orchestration code stays in ingress/session crates. [socket2](https://crates.io/crates/socket2) provides the
socket-level surface for this path. [nix](https://crates.io/crates/nix) remains Layer 9 baseline for
signal handling, and general socket options. macOS and
Windows ICMP paths are FC-deferred per
[scope](../scope.md). Evidence:

- [dependency-decisions](../dependency-decisions.md) Layer 9
- [ingress/icmp_linux](../../s1/atoms/ingress/icmp_linux.md)
- [ingress/icmp_posix](../../s1/atoms/ingress/icmp_posix.md)
- [platform-substrates](../../s1/catalogs/cross-cutting/platform-substrates.md)
  ICMP Proxy matrix

## Consequences

- S3.6 ingress implementation must use [socket2](https://crates.io/crates/socket2)-based ICMP raw socket plumbing
  through the dedicated unsafe crate boundary. No ICMP library may be introduced
  without a new ADR.
- [nix](https://crates.io/crates/nix) usage stays for signals and general socket options,
  but ICMP raw socket code must not use [nix](https://crates.io/crates/nix).
- Platform-specific ICMP behavior for macOS and Windows remains FC-deferred.
