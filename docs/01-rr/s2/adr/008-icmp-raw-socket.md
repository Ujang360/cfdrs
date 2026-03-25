# ADR-008 - ICMP Raw Socket

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Decided |
| Origin | Layer 9 in [dependency-decisions](../dependency-decisions.md) |
| Risk linkage | R2.6 in [risks](../risks.md) |

## Question

Should cfdrs implement ICMP proxying using a third-party library
or a custom implementation using nix raw sockets?

## Decision

Custom implementation using nix raw sockets. Same build-not-buy
logic as ADR-007 (SOCKS5).

## Rationale and Evidence

Linux ICMP path requires `SOCK_DGRAM` unprivileged sockets,
ping-group detection via `/proc/sys/net/ipv4/ping_group_range`, per-platform
echo-ID tracking, and integration with the datagram session packet router. No
library wraps this surface correctly for the cfdrs deployment model. The Go
implementation is self-contained in `ingress/icmp_linux.go` using raw OS
primitives. The nix crate is already decided in Layer 9 for `sched_setaffinity`,
socket options, and signal handling — ICMP raw sockets are a natural extension
of that same capability. macOS and Windows ICMP paths are FC-deferred per
[scope](../scope.md). Evidence:

- [dependency-decisions](../dependency-decisions.md) Layer 9
- [ingress/icmp_linux](../../s1/atoms/ingress/icmp_linux.md)
- [ingress/icmp_posix](../../s1/atoms/ingress/icmp_posix.md)
- [platform-substrates](../../s1/catalogs/cross-cutting/platform-substrates.md)
  ICMP Proxy matrix

## Consequences

- S3.6 ingress implementation must use nix raw socket primitives
  directly. No ICMP library may be introduced without a new ADR.
  Platform-specific ICMP behavior for macOS and Windows remains FC-deferred.
