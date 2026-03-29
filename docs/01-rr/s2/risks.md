# S2.3 - Risks

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| S2.3 status | Closed |
| Consumes | [scope](scope.md), [dependency-decisions](dependency-decisions.md), [porting-friction](../s1/catalogs/cross-cutting/porting-friction/README.md), [concurrency](../s1/catalogs/cross-cutting/concurrency/README.md), [error-propagation](../s1/catalogs/cross-cutting/error-propagation/README.md), [init-teardown](../s1/catalogs/cross-cutting/init-teardown/README.md), [wire-protocol](../s1/catalogs/cross-cutting/wire-protocol/README.md), [audit-analysis](../s1/audit-analysis.md) |
| Produces | Inputs to [adr](adr/README.md) S2.4, [invariants](invariants.md) S2.5, and S2.6 architecture |

This document converts S1 cross-cutting evidence and S2.1/S2.2 constraints into
an implementation risk matrix for S3 planning.

## Method

Risk scoring uses a 3x3 matrix:

- Probability: Low / Medium / High
- Impact: Low / Medium / High
- Priority:
  - P0 = High x High
  - P1 = High x Medium or Medium x High
  - P2 = Medium x Medium or High x Low or Low x High
  - P3 = Low x Medium or Low x Low

Each risk must map to at least one S1 source and one mitigation action.

## Risk Domains

| Domain | Source catalogs | Notes |
| --- | --- | --- |
| R1 Porting Friction | [porting-friction](../s1/catalogs/cross-cutting/porting-friction/README.md) | 11 friction categories with ranked severity |
| R2 Concurrency | [concurrency](../s1/catalogs/cross-cutting/concurrency/README.md), [shared-state](../s1/catalogs/domain/shared-state.md) | Actor topology, spawn sites, channel behavior |
| R3 Error Propagation | [error-propagation](../s1/catalogs/cross-cutting/error-propagation/README.md) | Classification trees and recovery boundaries |
| R4 Init/Teardown | [init-teardown](../s1/catalogs/cross-cutting/init-teardown/README.md) | Startup DAG and shutdown sequencing |
| R5 Wire Protocol | [wire-protocol](../s1/catalogs/cross-cutting/wire-protocol/README.md) | Handshake, framing, and transport state machines |
| R6 Dependency/Scope | [dependency-decisions](dependency-decisions.md), [scope](scope.md) | ADR set, pinned decisions, and scope boundaries |

## Risk Matrix

| ID | Domain | Risk | Evidence | Probability | Impact | Priority | Mitigation / follow-up |
| --- | --- | --- | --- | --- | --- | --- | --- |
| R1.1 | Porting Friction | Implicit interface satisfaction gaps cause late trait design churn across tunnel core atoms | [porting-friction](../s1/catalogs/cross-cutting/porting-friction/README.md) Rank 1 | High | High | P0 | In S2.6, lock trait boundary map before crate slicing; add interface inventory appendix keyed to atoms |
| R1.2 | Porting Friction | `context.Context` translation splits into multiple Rust mechanisms and destabilizes call signatures | [porting-friction](../s1/catalogs/cross-cutting/porting-friction/README.md) Rank 2 | High | High | P0 | Contract decision in [ADR-019](adr/019-cancellation-timeout-propagation-contract.md); enforce one preferred token propagation pattern in S2.6 |
| R1.3 | Porting Friction | Error type-switch sites become non-exhaustive Rust mappings | [porting-friction](../s1/catalogs/cross-cutting/porting-friction/README.md) Rank 3 | Medium | High | P1 | Define top-level error enum in S3.0; map each known switch branch explicitly |
| R1.4 | Porting Friction | Build-tag and feature-gated branches drift from Linux-only RR assumptions | [runtime-pattern-friction](../s1/catalogs/cross-cutting/porting-friction/runtime-pattern-friction.md), [scope](scope.md) | Medium | Medium | P2 | Keep Linux-only CI profile in S3; add compile-time check table in S2.9 |
| R1.5 | Porting Friction | `defer`-based cleanup and panic-recovery behavior changes alter teardown parity | [runtime-pattern-friction](../s1/catalogs/cross-cutting/porting-friction/runtime-pattern-friction.md) | Medium | Medium | P2 | Require explicit shutdown and drop semantics in critical loops; parity-test teardown paths in S4 |
| R1.6 | Porting Friction | `CustomDuration` serializes as integer seconds in JSON but as Go duration strings in YAML; a config round-trip through the wrong format silently produces a different timeout value — a parity failure with no error signal | [porting-friction](../s1/catalogs/cross-cutting/porting-friction/README.md) | Medium | High | P1 | Serialization contract in [ADR-013](adr/013-customduration-dual-format-contract.md); enforce dual-path round-trip tests before S3.1 |
| R2.1 | Concurrency | Unbounded stream/task fan-out under QUIC leads to scheduler pressure and memory spikes | [concurrency](../s1/catalogs/cross-cutting/concurrency/README.md) Quirk: goroutine per stream | High | High | P0 | Introduce bounded concurrency guard in transport accept loop; size caps in S2.6 architecture |
| R2.2 | Concurrency | Datagram v3 session migration introduces stale cancellation bindings and leaked session tasks | [concurrency](../s1/catalogs/cross-cutting/concurrency/README.md) Quirk: session migration | Medium | High | P1 | Lifecycle contract in [ADR-016](adr/016-session-migration-lifecycle.md); enforce explicit migration state machine and token rebinding rules |
| R2.3 | Concurrency | Channel capacity mismatches change close behavior and deadlock characteristics | [concurrency](../s1/catalogs/cross-cutting/concurrency/channels-select.md) | Medium | Medium | P2 | Track semantic buffer sizes per channel in architecture notes; add parity tests for close races |
| R2.4 | Concurrency | Fire-and-forget management goroutines accumulate on slow links | [concurrency](../s1/catalogs/cross-cutting/concurrency/README.md) Quirk: ping goroutine | Medium | Medium | P2 | Use bounded periodic tasks with timeout and cancellation in management service |
| R2.5 | Concurrency | Shared mutable registries diverge under HA load due to incorrect atomic/lock choices | [shared-state](../s1/catalogs/domain/shared-state.md), [concurrency](../s1/catalogs/cross-cutting/concurrency/patterns-primitives.md) | Medium | High | P1 | Declare lock/atomic policy per registry in S2.6; enforce with crate-level wrappers |
| R2.6 | Concurrency | `quiche::Connection` is `!Send` due to BoringSSL's thread-local error queue; connections must remain on their birth thread and a [tokio](https://crates.io/crates/tokio) task touching a connection from the wrong thread panics at runtime — invisible at compile time | [dependency-decisions](dependency-decisions.md) § 1.1 and § 3.1, [concurrency](../s1/catalogs/cross-cutting/concurrency/README.md) | High | High | P0 | Policy decision in [ADR-018](adr/018-quic-connection-ownership-enforcement.md); enforce wrappers and boundary constraints in S2.6 |
| R3.1 | Error Propagation | Misclassification of connection/supervisor error classes causes wrong retry vs abort outcomes | [error-propagation](../s1/catalogs/cross-cutting/error-propagation/README.md) | High | High | P0 | Taxonomy policy in [ADR-012](adr/012-error-taxonomy-and-recoverability-policy.md); implement shared mapping in S3.0 foundation crate |
| R3.2 | Error Propagation | Sentiment/string-based checks produce brittle behavior under message wording differences | [error-propagation/recovery-absorption](../s1/catalogs/cross-cutting/error-propagation/recovery-absorption.md) | Medium | Medium | P2 | Replace string matching with typed classifiers wherever possible; isolate unavoidable string paths |
| R3.3 | Error Propagation | Panic-recovery boundaries differ from Go and hide transport teardown errors | [error-propagation/recovery-absorption](../s1/catalogs/cross-cutting/error-propagation/recovery-absorption.md) | Medium | High | P1 | Define explicit panic/abort policy and per-boundary handling rules before implementation |
| R3.4 | Error Propagation | Datagram/session close errors lose severity metadata and downgrade observability | [error-propagation](../s1/catalogs/cross-cutting/error-propagation/README.md) session-layer section | Medium | Medium | P2 | Preserve severity-bearing error types in session layer; ensure logger mapping is lossless |
| R3.5 | Error Propagation | RPC retryable/non-retryable edge is implemented inconsistently across clients | [error-propagation](../s1/catalogs/cross-cutting/error-propagation/README.md) RPC-boundary section | Medium | High | P1 | Centralize RPC error mapping in one module; forbid ad-hoc per-call retry decisions |
| R4.1 | Init/Teardown | Startup DAG ordering drift breaks config, logger, observer, and orchestrator assumptions | [init-teardown](../s1/catalogs/cross-cutting/init-teardown/README.md) Initialization DAG | Medium | High | P1 | Ordering contract in [ADR-014](adr/014-startup-dag-contract.md); encode startup phases explicitly in bootstrap APIs |
| R4.2 | Init/Teardown | Graceful shutdown hangs if one actor misses graceful signal branch | [init-teardown/shutdown-teardown](../s1/catalogs/cross-cutting/init-teardown/shutdown-teardown.md) | Medium | High | P1 | Teardown contract in [ADR-015](adr/015-graceful-shutdown-contract.md); require two-phase graceful then hard-cancel behavior |
| R4.3 | Init/Teardown | `init()` side effects and static registration timing differ in Rust startup path | [porting-friction](../s1/catalogs/cross-cutting/porting-friction/README.md), [init-teardown/startup-sequence](../s1/catalogs/cross-cutting/init-teardown/startup-sequence.md) | Medium | Medium | P2 | Move all global init into explicit bootstrap step; avoid hidden static side effects |
| R4.4 | Init/Teardown | Hot-reload and proxy handoff sequencing causes transient request disruption | [init-teardown](../s1/catalogs/cross-cutting/init-teardown/README.md), [scope](scope.md) | Low | High | P2 | Maintain copy-on-write proxy swap semantics and explicit drain windows |
| R5.1 | Wire Protocol | Protocol-selection and fallback logic deviates from Go edge negotiation | [wire-protocol/transport-handshake](../s1/catalogs/cross-cutting/wire-protocol/transport-handshake.md), [scope](scope.md) QUIC-only | Medium | High | P1 | Keep QUIC-only RR behavior explicit; document fallback exclusion in parity contracts |
| R5.2 | Wire Protocol | Control-plane RPC framing differences break registration/config flows | [wire-protocol/rpc-datagram](../s1/catalogs/cross-cutting/wire-protocol/rpc-datagram.md) | Medium | High | P1 | Validate Cap'n Proto request/response frames against oracle captures before S3.2 completion |
| R5.3 | Wire Protocol | Datagram v2/v3 format and migration semantics diverge under mixed traffic | [wire-protocol/rpc-datagram](../s1/catalogs/cross-cutting/wire-protocol/rpc-datagram.md), [concurrency](../s1/catalogs/cross-cutting/concurrency/README.md) | Medium | High | P1 | Protocol and migration contract in [ADR-016](adr/016-session-migration-lifecycle.md); enforce v2 or v3 compatibility matrix in staged tests |
| R5.4 | Wire Protocol | Management websocket/auth wire behavior drifts from expected event contract | [wire-protocol](../s1/catalogs/cross-cutting/wire-protocol/README.md), [error-propagation](../s1/catalogs/cross-cutting/error-propagation/README.md) | Low | Medium | P3 | Reuse shared event schema and closure classification helpers in management layer |
| R6.1 | Dependency/Scope | Divergent metrics API usage across crates causes wide observability rework | [dependency-decisions](dependency-decisions.md), [ADR-003](adr/003-metrics-facade-vs-direct-prometheus.md) | Medium | Medium | P2 | Apply [ADR-003](adr/003-metrics-facade-vs-direct-prometheus.md) before S3 metrics instrumentation starts |
| R6.2 | Dependency/Scope | Parser strategy drift creates wire-path performance and correctness churn | [dependency-decisions](dependency-decisions.md), [ADR-004](adr/004-parser-strategy-combinator-vs-manual.md) | Medium | Medium | P2 | Apply [ADR-004](adr/004-parser-strategy-combinator-vs-manual.md) before S3.2 and S3.3 transport and RPC implementation |
| R6.3 | Dependency/Scope | Retry strategy drift yields inconsistent backoff semantics | [dependency-decisions](dependency-decisions.md), [ADR-005](adr/005-retry-backoff-library-vs-custom.md) | Low | Medium | P3 | Apply [ADR-005](adr/005-retry-backoff-library-vs-custom.md) before S3.5 supervisor or orchestration stage |
| R6.4 | Dependency/Scope | Management stack drift delays management and auth implementation | [dependency-decisions](dependency-decisions.md), [ADR-006](adr/006-management-http-stack.md) | Low | Medium | P3 | Apply [ADR-006](adr/006-management-http-stack.md) before S3.7 management stage |
| R6.5 | Dependency/Scope | QUIC dependency stack and allocator constraints force late transport pivot | [dependency-decisions](dependency-decisions.md), [scope](scope.md) | Medium | High | P1 | Keep fallback plan documented; lock allocator strategy in S2.9 guardrails |
| R6.6 | Dependency/Scope | Scope boundaries (HTTP/2 deferred, platform exclusions) leak into parity expectations | [scope](scope.md), [docs/01-rr/README.md](../README.md) | Medium | Medium | P2 | Access parity boundary codified in [ADR-017](adr/017-access-s4-drop-in-contract.md); encode remaining scope exclusions in parity TOML design (S2.7) and S4 sign-off criteria |

## Critical Path Risk Concentration

Critical-path atoms from [audit-analysis](../s1/audit-analysis.md) (membership >=12)
are overlaid below.

| Critical-path atom | Concentrated risks | Implication |
| --- | --- | --- |
| [tunnel/configuration](../s1/atoms/cmd/cloudflared/tunnel/configuration.md) | R4.1, R6.6 | Startup and scope-boundary contracts must be frozen before S3.1 |
| [connection/control](../s1/atoms/connection/control.md) | R1.1, R3.1, R5.2 | Control stream and error taxonomy require early interface and framing lock |
| [tunnel/cmd](../s1/atoms/cmd/cloudflared/tunnel/cmd.md) | R4.1, R4.2 | Bootstrap and shutdown ordering are high-impact for first executable binary |
| [connection/protocol](../s1/atoms/connection/protocol.md) | R5.1, R6.5, R6.6 | Transport policy must remain QUIC-only and dependency-stable in RR |
| [management/service](../s1/atoms/management/service.md) | R2.4, R5.4, R6.4 | Management stack choice and goroutine policy need early closure |
| [quic/v3/session](../s1/atoms/quic/v3/session.md) | R2.2, R3.4, R5.3 | Session lifecycle is a parity hotspot across concurrency, errors, and framing |
| [supervisor/tunnel](../s1/atoms/supervisor/tunnel.md) | R2.1, R3.1, R4.2 | Retry/recover and shutdown behavior remain top-risk in tunnel core |

## S2.3 Exit Gate

| Criterion | Status |
| --- | --- |
| Every risk has direct S1 or S2 evidence links | Yes |
| Probability x impact assigned for every risk | Yes |
| Mitigation or acceptance strategy recorded for every risk | Yes |
| ADR-linked dependency risks mapped (ADR-003/004/005/006) | Yes |
| Critical-path overlay includes all 7 load-bearing atoms | Yes |
| Scope-boundary risks from S2.2 are explicitly represented | Yes |
| Matrix is coherent with S2.1 dependency decisions | Yes |

## Notes

- This register is intentionally implementation-facing: each risk is phrased as
  a concrete failure mode that can be tested or gated in S3/S4.
- Risk priorities are planning guidance for S2.4-S2.8, not production incident
  severity classes.
