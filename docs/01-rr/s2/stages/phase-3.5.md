# Phase 3.5 — Configuration Runtime and Supervisor

| | |
| --- | --- |
| Stage | 3.5 |
| Scope | Orchestrator, config authority negotiation, HA supervisor |
| Entry condition | Stage 3.1 exit gate passed AND stage 3.3 exit gate passed AND stage 3.4 exit gate passed |
| Exit condition | 1 Must-tier contract green |
| Context risk | High (~104K practical) — split into 2 sub-stages; Extended agent recommended |

---

## Crates

| Order | Crate | Group | Must | Should | Total |
| --- | --- | --- | --- | --- | --- |
| 1 | `config-runtime` | config/ | 0 | 17 | 17 |
| 2 | `tunnel-supervisor` | tunnel/ | 1 | 2 | 3 |

### Implementation order

`config-runtime` first — the orchestrator must exist before the
supervisor can consume it. Depends on `config-core` (3.1),
`tunnel-core` (3.0), `common-signal`, `common-observability`.

`tunnel-supervisor` second — depends on `tunnel-connection` (3.3),
`common-retry`, `common-signal`, `common-observability`. Consumes
`config-runtime` indirectly via `ConfigManager` trait.

### Intra-stage dependency chain

```text
config-runtime   (imports config-core, tunnel-core, common-signal,
                  common-observability)

tunnel-supervisor (imports tunnel-core, tunnel-connection,
                   common-retry, common-signal, common-observability)
```

No direct dependency between these two crates — they can be
implemented in parallel sessions if the `ConfigManager` trait
from `tunnel-core` (3.0) is stable.

---

## Parity Gate

**Must-tier contracts (1):**

- `LIFE.protocol_fallback_state_machine` — QUIC→HTTP2 fallback +
  reset (in `tunnel-supervisor`)

**Should-tier contracts (19):**

- `CONFIG.update_configuration` — orchestrator update
- `CONFIG.override_warp_routing` — local authority override
- `CONFIG.concurrent_update_and_read` — concurrent safety
- 14 additional config-runtime contracts
- `LIFE.protocol_fallback_*` — 2 additional supervisor contracts

Must gate is low (1 contract) because behavioral complexity here
is primarily in config negotiation, which is Should-tier.

---

## Key Deliverables

- `Orchestrator` with versioned config swap, start-before-stop proxy
  hot-swap, `overrideRemoteWarpRoutingWithLocalValues`
- `ConfigManager` implementation (satisfies `tunnel-core::ConfigManager`)
- File watcher event loop (inotify → `ConfigDidUpdate`)
- `FeatureSelector` and `FeatureFlags` — DNS TXT periodic refresh,
  FNV hash percentile rollout
- Supervisor fan-out (start connection 0, wait for `connectedSignal`,
  stagger 1..N-1)
- `tunnelErrors` channel fan-in
- Protocol selection and PQ fallback (`selectNextProtocol`)
- `ConnTracker`, `ConnectedFuse` one-shot signal
- HA slot management (`tunnelsForHA`)

---

## Invariants Active

| ID | Invariant | Enforcement |
| --- | --- | --- |
| PROXY-3 | Config update preserves local authority | `CONFIG.override_warp_routing` |
| CONFIG-2 | Concurrent config reads are safe | `CONFIG.concurrent_update_and_read` |
| LIFE-3 | Protocol fallback state machine parity | `LIFE.protocol_fallback_state_machine` |
| ARCH-6 | Cancellation propagation through supervisor | Token passing |

---

## Risks

| Risk | Description | Mitigation |
| --- | --- | --- |
| R4.1 | Startup DAG ordering drift | ADR-014 explicit phase ordering |
| R4.2 | Shutdown race between supervisor and connections | ADR-015 two-phase shutdown contract |
| R2.5 | Shared mutable registries under HA load | Lock/atomic policy per registry |

---

## Session Guidance

- **Session count:** 2 sessions (one per crate)
- **Cognitive load:** Moderate — orchestrator is a continuous event
  loop with multiple config change sources; supervisor manages N
  concurrent connections with backoff coordination
- **Critical path:** On the critical path (3.4 → **3.5** → 3.6)
- **ractor usage:** Both `config-runtime` (orchestrator supervision
  tree) and `tunnel-supervisor` (HA supervision) use ractor actors
- **Cargo check:** `cargo check --workspace` after each crate
