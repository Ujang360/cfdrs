# Phase 3.8 — Application and CLI

| | |
| --- | --- |
| Stage | 3.8 |
| Scope | Binary assembly, CLI command tree, startup DAG, shutdown |
| Entry condition | All prior stages (3.0–3.7) exit gates passed |
| Exit condition | All 9 Must-tier contracts green |

---

## Crates

| Order | Crate | Group | Must | Should | Total |
| --- | --- | --- | --- | --- | --- |
| 1 | `operator-cli-common` | operator/ | 0 | 0 | 0 |
| 2 | `operator-cli-native` | operator/ | 6 | 2 | 8 |
| 3 | `operator-cli-compat` | operator/ | 0 | 0 | 0 |
| 4 | `operator-cli` | operator/ | 0 | 0 | 0 |
| 5 | `app` | app/ | 3 | 0 | 3 |

### Implementation order

`operator-cli-common` first — shared flag types and formatters.
No contracts.

`operator-cli-native` second — full clap derive command tree, flag
definitions, env-var bindings. 6 Must-tier contracts.

`operator-cli-compat` third — FC-deferred skeleton only. No logic.

`operator-cli` fourth — feature-flag diamond resolver. No logic
of its own, just re-exports.

`app` last — wires all 25 other crates together. Startup DAG,
shutdown sequence, overwatch service registry.

### Intra-stage dependency chain

```text
operator-cli-common
  ↑                ↑
operator-cli-native  operator-cli-compat (skeleton)
  ↑                ↑
      operator-cli

app  (imports operator-cli + all tunnel/config/host/common crates)
```

---

## Parity Gate

**Must-tier contracts (9):**

`operator-cli-native` (6):

- `PROXY.tunnel_tag_parsing` — tunnel tag parsing
- `PROXY.tunnel_cmd` — tunnel command dispatch
- `CONFIG.tunnel_cmd_config` — tunnel command configuration
- `PROXY.tunnel_subcommand_context` — subcommand context
- 2 additional CLI contracts

`app` (3):

- `LIFE.manager_add_remove` — service lifecycle add/remove
- `LIFE.manager_duplicate` — duplicate service handling
- `LIFE.manager_error_channel` — error channel propagation

All 9 must be `green` before stage 3.9 begins.

---

## Key Deliverables

### `operator-cli-common`

- Shared flag types, output formatting helpers
- Error wrappers, build info display
- Deprecated command handling, logger construction

### `operator-cli-native`

- Full command tree: tunnel run/create/delete/list/info/token/route/
  cleanup/ingress, access login/curl/ssh/rdp/smb/tcp, service
  install/uninstall, proxydns, tail, version, management token
- Flag definitions and env-var bindings
- Tunnel subcommand context, credential finder
- Quick tunnel provisioning
- stdin control (`--stdin-control`, `reconnect [delay]`)

### `operator-cli-compat`

- Skeleton crate only — activated if S4 reveals CLI flag divergence

### `operator-cli`

- Feature-flag diamond resolver (`#[cfg(feature = "native")]` vs
  `#[cfg(feature = "compat")]`)

### `app`

- `main.rs` — binary entry point
- `startup.rs` — 13-phase startup DAG per
  [ADR-014](../../adr/014-startup-dag-contract.md)
- `shutdown.rs` — two-phase shutdown per
  [ADR-015](../../adr/015-graceful-shutdown-contract.md)
- `overwatch.rs` — `AppManager` plugin-style service registry
- `registry.rs` — `prometheus::Registry` construction
- `wire.rs` — dependency injection

---

## Invariants Active

| ID | Invariant | Enforcement |
| --- | --- | --- |
| LIFE-1 | Startup DAG ordering | `LIFE.manager_*` contracts, ADR-014 |
| LIFE-2 | Graceful shutdown sequence | ADR-015 shutdown contracts |
| ARCH-6 | Cancellation propagation end-to-end | Full integration |

---

## Risks

| Risk | Description | Mitigation |
| --- | --- | --- |
| R4.1 | Startup DAG ordering drift at integration | ADR-014 explicit phase ordering |
| R6.1 | Scope creep at assembly stage | Strict scope from prior stages |

---

## Session Guidance

- **Session count:** 3–5 sessions (cli-common + cli-native may
  combine; app is its own session)
- **Cognitive load:** Moderate for CLI crates; high for `app` which
  must integrate all 25 crates with correct startup/shutdown ordering
- **Integration testing:** First stage where cross-crate behavioral
  tests are meaningful — use `app/tests/integration/`
- **Cargo check:** Full `cargo check --workspace` and `cargo build`
  — this is the first stage producing a runnable binary
