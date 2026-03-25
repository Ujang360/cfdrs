# ADR-014 - Startup DAG Contract

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Decided |
| Origin | R4.1 in [risks](../risks.md) |
| Risk linkage | R4.1 (P1) |

## Question

How should cfdrs encode startup ordering so bootstrap dependencies remain deterministic across config, logging, observer, transport, and orchestration paths?

## Decision

Define startup as an explicit DAG with named phases and dependency edges. Startup order must be validated at bootstrap boundaries, not inferred from module initialization side effects.

## Alternatives Considered

- Implicit startup order via ad-hoc call sequencing.
- Explicit startup DAG contract with phase validation.

## Rationale and Evidence

- [risks](../risks.md) records R4.1: startup ordering drift can break core runtime assumptions.
- [init-teardown catalog](../../s1/catalogs/cross-cutting/init-teardown/README.md) identifies startup sequence as a cross-cutting behavioral surface.
- [tunnel/configuration atom](../../s1/atoms/cmd/cloudflared/tunnel/configuration.md) is a critical-path coordination point.
- Explicit phase ordering reduces accidental regressions and supports parity testability.

## Consequences

- S2.6 architecture must publish a startup phase graph and boundary ownership.
- S3 bootstrap code must assert phase order and fail fast on invalid sequencing.
- S4 parity verification should include startup-order-sensitive scenarios.

## Scope Notes

- This ADR governs startup order, not shutdown behavior.
- Signal handling and graceful teardown are covered separately by [ADR-015](015-graceful-shutdown-contract.md).
