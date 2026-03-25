# ADR-012 - Error Taxonomy and Recoverability Policy

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Decided |
| Origin | Layer 5 in [dependency-decisions](../dependency-decisions.md), R3.1 in [risks](../risks.md) |
| Risk linkage | R3.1 (P0), R3.5 (P1) |

## Question

How should [cfdrs](../../../../README.md) represent cross-layer errors so retry, abort, and escalation behavior stays consistent with S1 semantics?

## Decision

Use a typed, shared error taxonomy with explicit recoverability classification at boundary layers. Ban boundary-level string matching and ad-hoc retry decisions.

## Alternatives Considered

- Keep loosely typed string- or message-based classification at call sites.
- Define shared typed error classes with centralized recoverability mapping.

## Rationale and Evidence

- [dependency-decisions](../dependency-decisions.md) identifies layered Go error behavior and already points to typed enums with recoverability semantics.
- [risks](../risks.md) marks R3.1 as P0 and R3.5 as P1 when classification diverges across modules.
- [error-propagation catalog](../../s1/catalogs/cross-cutting/error-propagation/README.md) shows multiple boundary sites with different retry expectations.
- Critical-path atoms [connection/control](../../s1/atoms/connection/control.md) and [supervisor/tunnel](../../s1/atoms/supervisor/tunnel.md) require stable retry and abort decisions.

## Consequences

- S2.6 must define one cross-layer error taxonomy boundary and ownership model.
- S3.0 foundation work must include central mapping from source errors to shared classes.
- S3 and S4 verification should include table-driven checks for recoverable versus fatal classification paths.

## Scope Notes

- This ADR sets policy and boundary requirements, not final enum naming.
- Panic and unwind boundary behavior remains a separate ADR candidate.
