# ADR-004 - Binary Parser Strategy

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Decided |
| Origin | Layer 2.6 in [dependency-decisions](../dependency-decisions.md) |
| Risk linkage | R6.2 in [risks](../risks.md) |

## Question

Should [cfdrs](https://github.com/Ujang360/cfdrs) parse wire formats with a combinator-based parser framework, or rely on ad-hoc manual byte-buffer slicing across each protocol path?

## Decision

Use combinator-based parsing as the default for custom wire formats. Allow manual parsing only in narrowly bounded hot paths when a measurable performance requirement is documented.

## Alternatives Considered

- Manual byte-buffer slicing everywhere.
- Combinator default with explicit exception process for specific hot paths.

## Rationale and Evidence

- [dependency-decisions](../dependency-decisions.md) already selects combinator parsing for datagram v2 or v3 frame types, QUIC stream preamble parsing, and Cap'n Proto frame boundary handling.
- [risks](../risks.md) records R6.2: unresolved parser strategy creates correctness and performance churn in transport and RPC stages.
- [wire-protocol](../../s1/catalogs/cross-cutting/wire-protocol/README.md) and [connection/protocol atom](../../s1/atoms/connection/protocol.md) indicate parsing behavior is cross-cutting and sensitive to framing consistency.
- A combinator-first policy gives uniform error reporting and testability while preserving escape hatches where profiling proves manual parsing is required.

## Consequences

- S2.6 architecture must define one parser module boundary and one exception policy for manual paths.
- S3 implementations must document and benchmark any manual parser exception.
- Parity harness design in phase 2.7 should include parser-focused fixtures for accepted and rejected frames.

## Scope Notes

- This ADR does not select a specific crate name.
- Cap'n Proto schema semantics remain governed by protocol contracts, not by parser style alone.
