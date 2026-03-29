# Phase 3.0 — Foundation

| | |
| --- | --- |
| Stage | 3.0 |
| Scope | Shared types, error taxonomy, trait interfaces |
| Entry condition | S2 exit gate passed (phase 2.10) |
| Exit condition | All 6 Must-tier contracts green |

---

## Crates

| Order | Crate | Group | Must | Should | Contracts |
| --- | --- | --- | --- | --- | --- |
| 1 | `common-error` | common/ | 0 | 0 | 0 |
| 2 | `common-wire-primitives` | common/ | 6 | 0 | 6 |
| 3 | `tunnel-core` | tunnel/ | 0 | 0 | 0 |

### Implementation order

`common-error` first — every other crate depends on it.
`common-wire-primitives` second — wire vocabulary types used across
all layers. `tunnel-core` third — pure trait hub, depends on
`common-wire-primitives` for shared types.

`common-error` and `common-wire-primitives` have no internal
dependency and can be implemented in parallel sessions.

---

## Parity Gate

**Must-tier contracts (6):**

- `TRANS.header_serialize_roundtrip` — HTTP header round-trip
- `TRANS.websocket_accept_key_rfc6455` — RFC 6455 key generation
- 4 additional wire-primitive contracts from `common-wire-primitives`

All 6 must be `green` before any stage 3.1, 3.2, or 3.3 crate begins.

---

## Key Deliverables

- Top-level `Error` enum with `Unrecoverable`/`Recoverable` split
  per [ADR-012](../../adr/012-error-taxonomy-and-recoverability-policy.md)
- `CustomDuration` dual-format serializer per
  [ADR-013](../../adr/013-customduration-dual-format-contract.md)
- `RequestID`, `CfTraceID`, `ConnectionIndex`, protocol event enums
- `OriginProxy`, `ConfigManager`, `TunnelStream`, `DatagramConn`
  traits in `tunnel-core`
- Worker boundary types: `TunnelWorkerHandle` (`!Send`),
  `ProxyWorkerHandle` (`!Send`), `SessionAssignment`, `WorkerIndex`

---

## Invariants Active

| ID | Invariant | Enforcement |
| --- | --- | --- |
| ARCH-1 | Proxy data fat enum — no `Box<dyn>` on hot path | Type definition in `tunnel-core` |
| ARCH-3 | Worker group boundary errors are `common-error` types only | `common-error` taxonomy |
| ARCH-4 | Fat enum dispatch — exhaustive match | Enum variants in `tunnel-core` |
| ERR-1 | No panics on malformed input in library crates | `common-error` discipline |

---

## Risks

| Risk | Description | Mitigation |
| --- | --- | --- |
| R1.1 | Trait design churn across tunnel core atoms | Lock trait interfaces before 3.1 starts |
| R1.3 | Error type-switch sites become non-exhaustive | Map each switch branch explicitly in `common-error` |
| R3.1 | Error misclassification causes wrong retry behavior | Centralize mapping in `common-error` |

---

## Session Guidance

- **Session count:** 2–3 sessions (one per crate)
- **Cognitive load:** Low — pure types and traits, no async, no I/O
- **Cargo check:** `cargo check --workspace` must pass after each crate
