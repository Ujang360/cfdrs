# ADR-005 - Retry and Backoff Strategy

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Decided |
| Origin | Layer 7 in [dependency-decisions](../dependency-decisions.md) |
| Risk linkage | R6.3 in [risks](../risks.md) |

## Question

Should [cfdrs](https://github.com/Ujang360/cfdrs) use a third-party async backoff library directly in runtime code, or maintain a small custom backoff implementation aligned to Go behavior?

## Decision

Use a custom backoff policy module for RR parity-critical retry semantics. A third-party library may be used internally only as an implementation helper if it does not leak API or behavior constraints.

## Alternatives Considered

- Use a third-party retry library as the primary runtime policy.
- Keep custom retry policy as the contract and optionally use helper utilities behind it.

## Rationale and Evidence

- [dependency-decisions](../dependency-decisions.md) notes the Go baseline has custom jitter and reset logic for retry behavior.
- [risks](../risks.md) records R6.3: unresolved retry strategy yields inconsistent backoff semantics, especially before supervisor and orchestration work.
- [supervisor/tunnel atom](../../s1/atoms/supervisor/tunnel.md) is on the critical path and directly depends on stable retry contracts.
- A custom policy boundary preserves behavioral control and parity while avoiding lock-in to a crate-specific policy model.

## Consequences

- S2.6 must define one retry policy interface with deterministic jitter and reset semantics.
- S3 supervisor and orchestration code must consume the shared policy, not ad-hoc retry loops.
- Phase 2.7 parity design should include explicit retry-sequence and reset-case fixtures.

## Scope Notes

- This ADR selects policy ownership, not a final internal helper crate.
- Retry observability hooks should align with the metrics boundary from [ADR-003](003-metrics-facade-vs-direct-prometheus.md).
