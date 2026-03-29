# Phase 3.7 — Management and Diagnostics

| | |
| --- | --- |
| Stage | 3.7 |
| Scope | Management HTTP/WebSocket service, system diagnostics |
| Entry condition | Stage 3.5 exit gate passed AND stage 3.6 exit gate passed |
| Exit condition | All 6 Must-tier contracts green |

---

## Crates

| Order | Crate | Group | Must | Should | Skip | Total |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | `tunnel-management` | tunnel/ | 6 | 21 | 0 | 27 |
| 2 | `host-diagnostic` | host/ | 0 | 11 | 1 | 12 |

### Implementation order

`tunnel-management` first — the management HTTP service must exist
before diagnostics handlers can be registered.

`host-diagnostic` second — provides system state collection and
diagnostic exposure. No dependency on `tunnel-management` but
logically pairs with it.

These two crates have no mutual dependency and can be implemented
in parallel sessions.

---

## Parity Gate

**Must-tier contracts (6):**

- `MGMT.disable_diagnostic_routes` — route disabling
- `MGMT.read_events_loop` — WebSocket streaming
- `MGMT.start_stream_*` — session limits (different actor,
  same actor)
- Additional management lifecycle contracts

**Skipped (1):** Windows diagnostic collector (`tracert`) — FC-deferred.

All 6 Must contracts must be `green` before stage 3.8 begins.

---

## Key Deliverables

### `tunnel-management`

- Management HTTP service (`/ping`, `/host_details`, `/logs` WebSocket)
- WebSocket log stream (start/stop_streaming, filters, idle 5min
  timeout, heartbeat 15s, 1 concurrent session, preemption,
  close codes 4001/4002/4003)
- Management access token middleware (`access_token` query param,
  JWT claims)
- `host_details` response (connector ID, optional IP via 1s TCP dial,
  hostname)

### `host-diagnostic`

- Linux system collector (`/proc/meminfo`, `/proc/cpuinfo`,
  `/proc/fd/`, `lsb_release`, `uname`)
- Log collectors (journalctl host path, Docker `docker logs`,
  Kubernetes `kubectl logs`)
- Network traceroute (Unix `traceroute` only)
- Diagnostic HTTP handlers (`/diag/system`, `/diag/tunnel`,
  `/diag/configuration`)
- Diagnostic zip bundler
- Diagnostic client (remote queries from `tunnel diagnose`)

---

## Invariants Active

| ID | Invariant | Enforcement |
| --- | --- | --- |
| MGMT-1 | One concurrent WebSocket session, preemption | `MGMT.start_stream_*` contracts |
| ERR-1 | No panics on malformed management input | Contract tests |

---

## Risks

| Risk | Description | Mitigation |
| --- | --- | --- |
| R2.4 | Fire-and-forget management goroutines | Bounded periodic tasks with timeout and cancellation |

---

## Session Guidance

- **Session count:** 2 sessions (one per crate)
- **Cognitive load:** Moderate — management WebSocket lifecycle has
  subtle sessionization and preemption logic; diagnostics is
  straightforward data collection
- **Off critical path:** 3.7 is not on the critical path; can
  overlap with late 3.6 work
- **Cargo check:** `cargo check --workspace` after each crate
