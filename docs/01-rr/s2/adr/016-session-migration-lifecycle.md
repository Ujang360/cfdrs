# ADR-016 - Session Migration Lifecycle Contract

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Decided |
| Origin | R2.2 and R5.3 in [risks](../risks.md) |
| Risk linkage | R2.2 (P1), R5.3 (P1) |

## Question

How should [cfdrs](https://github.com/Ujang360/cfdrs) model datagram session migration so token rebinding, cancellation, and v2 or v3 protocol behavior remain consistent under mixed traffic?

## Decision

Define an explicit session lifecycle state machine with migration states and token rebinding rules. Migration must be treated as a first-class state transition, not an implicit side effect.

## Alternatives Considered

- Best-effort migration through ad-hoc task updates.
- Explicit state machine with transition guards and cancellation rebinding.

## Rationale and Evidence

- [risks](../risks.md) marks R2.2 and R5.3 as P1 with concurrency and wire-protocol drift exposure.
- [concurrency catalog](../../s1/catalogs/cross-cutting/concurrency/README.md) documents session migration fragility.
- [wire-protocol catalog](../../s1/catalogs/cross-cutting/wire-protocol/README.md) highlights v2 or v3 compatibility requirements.
- [quic/v3/session atom](../../s1/atoms/quic/v3/session.md) is a critical-path parity hotspot.

## Consequences

- S2.6 must specify lifecycle states, transitions, and ownership boundaries.
- S3 session code must centralize migration logic and forbid out-of-band token rebinding.
- Phase 2.7 parity design should include migration fixtures for stale-token and mixed-traffic paths.

## Scope Notes

- This ADR defines lifecycle contract requirements, not final state enum names.
- Transport fallback policy remains outside this ADR.
