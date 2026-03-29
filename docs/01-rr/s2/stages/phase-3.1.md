# Phase 3.1 — Independent Crates

| | |
| --- | --- |
| Stage | 3.1 |
| Scope | Standalone crates with no tunnel behavioral dependencies |
| Entry condition | Stage 3.0 exit gate passed (6/6 Must green) |
| Exit condition | All 6 Must-tier contracts green |

---

## Crates

| Order | Crate | Group | Must | Should | Skip | Fuzz | Total |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | `common-signal` | common/ | 2 | 0 | 0 | 0 | 2 |
| 2 | `common-retry` | common/ | 0 | 6 | 0 | 0 | 6 |
| 3 | `common-observability` | common/ | 0 | 13 | 0 | 1 | 14 |
| 4 | `common-cfapi` | common/ | 0 | 7 | 11 | 0 | 18 |
| 5 | `common-token` | common/ | 2 | 18 | 0 | 0 | 20 |
| 6 | `config-core` | config/ | 2 | 0 | 0 | 0 | 2 |
| 7 | `tunnel-metrics` | tunnel/ | 0 | 2 | 0 | 0 | 2 |

### Implementation order

`common-signal` first — imported by `common-retry` and many
downstream crates. `common-retry` second — depends on `common-error`
and `common-signal`.

The remaining 5 crates (`common-observability`, `common-cfapi`,
`common-token`, `config-core`, `tunnel-metrics`) have no mutual
dependencies and can be implemented in any order or in parallel.

### Intra-stage dependency chain

```text
common-signal
  ↑
common-retry

common-observability   (independent)
common-cfapi           (independent)
common-token           (independent)
config-core            (independent, imports common-wire-primitives from 3.0)
tunnel-metrics         (independent, imports common-observability)
```

---

## Parity Gate

**Must-tier contracts (6):**

- `LIFE.signal_double_notify` — safe double-close on cancellation
- `LIFE.signal_wait` — signal wait semantics
- `CRED.credentials_read` — credential file resolution
- `CRED.credentials_client` — credential client factory
- `CONFIG.yaml_file_settings` — YAML config deserialization
- `CONFIG.origin_request_roundtrip` — origin request config round-trip

All 6 must be `green` before stage 3.8 crates begin. Stages 3.2 and
3.3 depend only on 3.0, not 3.1 — they can proceed in parallel.

---

## Key Deliverables

- `CancellationToken` wrapper, `ShutdownPhase` enum, `GracefulShutdownC`
- `BackoffHandler` with exponential delay, jitter, `Recoverable<E>`
  typestate
- `MetricsRegistrar` trait, counter/gauge/histogram macros
- `RESTClient` with `fetchExhaustively` pagination
- Token acquisition lifecycle, credential file operations, browser
  launch, origin certificate codec
- Config file discovery, two-pass YAML decode, `Configuration` struct,
  4-layer config merge
- GCRA rate limiter via `governor`, `/metrics` + `/ready` endpoints

---

## Invariants Active

| ID | Invariant | Enforcement |
| --- | --- | --- |
| ARCH-6 | Cancellation propagation — every async entry carries token | `common-signal` contract |
| CONFIG-1 | `CustomDuration` round-trip parity | `config-core` dual-format tests |
| CONFIG-3 | YAML config deserialization parity | `config-core` tests |
| CRED-1 | Credential discovery order preserved | `common-token` tests |
| ERR-3 | Backoff retries, grace period, max duration | `common-retry` contracts |

---

## Risks

| Risk | Description | Mitigation |
| --- | --- | --- |
| R1.6 | `CustomDuration` JSON/YAML serialization asymmetry | Dual-path round-trip tests in `config-core` |
| R1.2 | `context.Context` translation for signal types | Follow ADR-019 token propagation pattern |

---

## Session Guidance

- **Session count:** 7 sessions (one per crate)
- **Cognitive load:** Low to moderate — each crate is self-contained
- **Parallelism:** After `common-signal` + `common-retry`, remaining
  5 crates can be implemented in parallel
- **Cargo check:** `cargo check --workspace` after each crate
