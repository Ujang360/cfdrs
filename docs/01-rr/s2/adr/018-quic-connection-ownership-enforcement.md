# ADR-018 - QUIC Connection Ownership Enforcement

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Decided |
| Origin | R2.6 in [risks](../risks.md), Layer 1.1 and 3.1 in [dependency-decisions](../dependency-decisions.md) |
| Risk linkage | R2.6 (P0) |

## Question

How should [cfdrs](../../../../README.md) enforce that QUIC connection objects, which are
`!Send`, never cross thread boundaries?

## Decision

Enforce QUIC connection ownership at the transport boundary using
`!Send` wrapper types and explicit ownership rules. No manual CPU
affinity (`sched_setaffinity`) is performed by cfdrs. `tokio-quiche`
manages its own thread scheduling internally. CPU affinity policy is
the operator's concern.

## Alternatives Considered

- Rely on runtime discipline without type-level enforcement.
- Enforce affinity by design with wrapper types and boundary constraints.

## Rationale and Evidence

- [risks](../risks.md) documents R2.6: `quiche::Connection` is `!Send`, cross-thread access can panic at runtime, and this risk is P0.
- [dependency-decisions](../dependency-decisions.md) sets a multi-thread runtime and QUIC as core transport, making affinity control mandatory.
- [quic/v3/session atom](../../s1/atoms/quic/v3/session.md) and [connection/protocol atom](../../s1/atoms/connection/protocol.md) are critical-path surfaces where misuse would be high impact.
- Type- and boundary-level enforcement reduces invisible runtime failure modes.

## Consequences

- S2.6 must define transport ownership boundaries that prevent cross-thread handle transfer.
- S3 transport and session code must route work to the owning thread rather than sharing raw connection handles.
- Parity and stress tests should include affinity misuse safeguards in phase 2.7 and S4 planning.

## Scope Notes

- This ADR governs ownership and scheduling boundaries, not CPU affinity policy for all tasks.
- Any future shift to a fully `Send` transport backend requires an ADR update.
