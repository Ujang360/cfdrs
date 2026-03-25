# ADR-001 - QUIC Transport

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Decided |
| Origin | Layer 3.1 in [dependency-decisions](../dependency-decisions.md) |
| Risk linkage | R6.5 in [risks](../risks.md) |

## Question

Should [cfdrs](../../../../README.md) use Cloudflare's tokio-integrated QUIC ([tokio-quiche](https://crates.io/crates/tokio-quiche))
or a pure-Rust QUIC stack as the primary transport?

## Decision

Use [tokio-quiche](https://crates.io/crates/tokio-quiche) as the primary QUIC transport. Zero-copy sends
are mandatory. gcongestion is mandatory (implied by zero-copy). Pure-Rust QUIC
is a conditional fallback only — activated if the [tokio-quiche](https://crates.io/crates/tokio-quiche) allocator link
conflict is unresolvable at S3 build time.

## Rationale and Evidence

[tokio-quiche](https://crates.io/crates/tokio-quiche) is Cloudflare's own production QUIC stack, already
integrated with [tokio](https://crates.io/crates/tokio), providing zero-copy sends and Google congestion control.
The fallback path (pure-Rust QUIC + FIPS-capable TLS) is pre-identified and
viable but not the active path. Evidence:

- [dependency-decisions](../dependency-decisions.md) § 3.1
- [wire-protocol](../../s1/catalogs/cross-cutting/wire-protocol/README.md)
- [connection/quic_connection](../../s1/atoms/connection/quic_connection.md)

## Consequences

- S2.6 must define the `cfdrs-tunnel-transport` abstraction seam
  to accommodate both paths without assuming QUIC-only. The allocator conflict
  check must occur on the first S3 build.
