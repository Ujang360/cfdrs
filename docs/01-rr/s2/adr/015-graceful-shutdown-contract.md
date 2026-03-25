# ADR-015 - Graceful Shutdown Contract

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Decided |
| Origin | R4.2 in [risks](../risks.md) |
| Risk linkage | R4.2 (P1) |

## Question

How should [cfdrs](https://github.com/Ujang360/cfdrs) standardize shutdown behavior so actors and transport tasks terminate deterministically under graceful and forced stop paths?

## Decision

Adopt a two-phase shutdown contract:

- Phase 1: graceful signal propagation with bounded wait.
- Phase 2: hard cancellation for remaining tasks after timeout.

All long-lived actors and service loops must implement both phases explicitly.

## Alternatives Considered

- Single-phase best-effort shutdown.
- Two-phase graceful-then-hard contract.

## Rationale and Evidence

- [risks](../risks.md) flags R4.2: shutdown hangs occur when one branch misses graceful signaling.
- [init-teardown shutdown catalog](../../s1/catalogs/cross-cutting/init-teardown/shutdown-teardown.md) identifies teardown sequencing as behavior-critical.
- [supervisor/tunnel atom](../../s1/atoms/supervisor/tunnel.md) and [tunnel/cmd atom](../../s1/atoms/cmd/cloudflared/tunnel/cmd.md) are high-impact shutdown surfaces.
- Two-phase semantics make cancellation behavior explicit and testable.

## Consequences

- S2.6 must define shared shutdown interfaces and timeout ownership.
- S3 actor and transport code must implement graceful and hard paths consistently.
- S4 parity checks should include interrupted and slow-shutdown scenarios.

## Scope Notes

- This ADR covers runtime shutdown behavior, not process supervisor policy outside RR.
- Panic and unwind behavior remains governed by [ADR-012](012-error-taxonomy-and-recoverability-policy.md) and later panic-boundary ADR work.
