# ADR-019 - Cancellation and Timeout Propagation Contract

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Decided |
| Origin | R1.2 in [risks](../risks.md) and porting-friction evidence |
| Risk linkage | R1.2 (P0) |

## Question

How should [cfdrs](../../../../README.md) represent Go `context.Context` behavior consistently across async boundaries in Rust?

## Decision

Define one standard propagation contract: explicit cancellation token plus explicit timeout value at EVL and actor boundaries. Do not use ad-hoc mixed patterns per module.

## Alternatives Considered

- Per-module mechanism choices with no shared contract.
- One contract with standardized boundary signatures and propagation behavior.

## Rationale and Evidence

- [risks](../risks.md) records R1.2 as P0: splitting `context.Context` behavior across multiple Rust mechanisms destabilizes signatures and behavior.
- [porting-friction catalog](../../s1/catalogs/cross-cutting/porting-friction/README.md) ranks context translation as a top friction source.
- [tunnel/configuration atom](../../s1/atoms/cmd/cloudflared/tunnel/configuration.md) and [connection/control atom](../../s1/atoms/connection/control.md) are central dispatch and control surfaces where inconsistent propagation would cascade.
- A single contract preserves parity semantics and reduces cross-crate signature churn before S2.6 architecture decisions.

## Consequences

- S2.6 must publish boundary signature rules for cancellation and timeout propagation.
- S3 implementations must reject new boundary APIs that omit the contract.
- Phase 2.7 parity design should include timeout and cancellation fixtures across registration, control stream, and session transitions.

## Scope Notes

- This ADR sets boundary contract requirements; it does not prescribe one concrete token type for every internal helper function.
- Context value bags are out of scope unless required by verified S1 behavior.
