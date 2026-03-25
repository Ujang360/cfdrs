# ADR-003 - Metrics API Boundary

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Decided |
| Origin | Layer 4.3 in [dependency-decisions](../dependency-decisions.md) |
| Risk linkage | R6.1 in [risks](../risks.md) |

## Question

Should [cfdrs](https://github.com/Ujang360/cfdrs) expose metrics through direct Prometheus client types, or through an internal facade API that hides backend-specific types from most crates?

## Decision

Adopt an internal metrics facade for crate-to-crate interfaces. Keep direct Prometheus client usage inside a dedicated metrics implementation boundary.

## Alternatives Considered

- Direct Prometheus usage everywhere.
- Metrics facade with Prometheus as the first backend.

## Rationale and Evidence

- [dependency-decisions](../dependency-decisions.md) marks observability blast radius as medium across connection, quic, supervisor, datagramsession, flow, ingress, and tunnelrpc.
- [risks](../risks.md) identifies R6.1: unresolved metrics API choice causes broad rework if selected late.
- Critical-path atoms in [audit-analysis](../../s1/audit-analysis.md) show cross-cutting interfaces must be stabilized before architecture and crate slicing.
- A facade reduces public API churn in S2.6 and S3 while preserving Prometheus compatibility required by S1 behavior.

## Consequences

- S2.6 must define one metrics facade module and one Prometheus adapter boundary.
- Hot-path crates depend on facade traits and value types, not exporter-specific types.
- Prometheus remains the default and required exporter in RR scope.

## Scope Notes

- This ADR chooses interface boundaries, not metric cardinality policy.
- Additional exporters are out of scope for RR and are not implied by this decision.
