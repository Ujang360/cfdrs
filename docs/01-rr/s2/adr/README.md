# S2.4 ADR Index

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Artifact | ADR corpus for phase 2.4 |

This directory stores phase 2.4 architecture decision records.

## Numbering Policy

- ADR numbers are stable once assigned.
- ADR-001 through ADR-011 use the original registry from
  [dependency-decisions](../dependency-decisions.md).
- New expanded S2.4 decisions start at ADR-012.
- ADR-007 and ADR-009 are resolved inline in
  [dependency-decisions](../dependency-decisions.md) and have no standalone files.
- ADR-018 and ADR-019 are the renamed successors of the previously
  misassigned ADR-010 and ADR-011 slots.

## ADR Files

| ADR | Title | Primary risks constrained | Status |
| --- | --- | --- | --- |
| [ADR-001](001-quic-transport.md) | QUIC transport | R6.5 | Decided |
| [ADR-002](002-fips-ci-build-matrix.md) | FIPS CI build matrix | — | Deferred — S4/S5 |
| [ADR-003](003-metrics-facade-vs-direct-prometheus.md) | Metrics API boundary | R6.1 | Decided |
| [ADR-004](004-parser-strategy-combinator-vs-manual.md) | Binary parser strategy | R6.2 | Decided |
| [ADR-005](005-retry-backoff-library-vs-custom.md) | Retry and backoff strategy | R6.3 | Decided |
| [ADR-006](006-management-http-stack.md) | Management HTTP stack | R6.4 | Decided |
| ADR-007 | SOCKS5 — build vs buy | — | Resolved inline |
| [ADR-008](008-icmp-raw-socket.md) | ICMP raw socket | R2.6 | Decided |
| ADR-009 | [sd-notify](https://crates.io/crates/sd-notify) — minimal vs full [systemd](https://crates.io/crates/systemd) crate | — | Resolved inline |
| [ADR-010](010-cli-compat.md) | CLI compatibility mode | — | Deferred — FC |
| [ADR-011](011-evl-architecture.md) | EVL architecture | — | Open — S2.6 |
| [ADR-012](012-error-taxonomy-and-recoverability-policy.md) | Error taxonomy and recoverability policy | R3.1 | Decided |
| [ADR-013](013-customduration-dual-format-contract.md) | CustomDuration dual-format contract | R1.6 | Decided |
| [ADR-014](014-startup-dag-contract.md) | Startup DAG contract | R4.1 | Decided |
| [ADR-015](015-graceful-shutdown-contract.md) | Graceful shutdown contract | R4.2 | Decided |
| [ADR-016](016-session-migration-lifecycle.md) | Session migration lifecycle contract | R2.2, R5.3 | Decided |
| [ADR-017](017-access-s4-drop-in-contract.md) | Access S4 drop-in replacement contract | R6.6 | Decided |
| [ADR-018](018-quic-thread-affinity-enforcement.md) | QUIC thread-affinity enforcement | R2.6 | Decided |
| [ADR-019](019-cancellation-timeout-propagation-contract.md) | Cancellation and timeout propagation contract | R1.2 | Decided |
