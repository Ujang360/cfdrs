# Phase 3.4 — Sessions

| | |
| --- | --- |
| Stage | 3.4 |
| Scope | UDP datagram sessions, muxer, ICMP router, session migration |
| Entry condition | Stage 3.3 exit gate passed (30/30 Must green) |
| Exit condition | All 52 Must-tier contracts green |
| Context risk | Critical (~168K practical) — split into 3 sub-stages mandatory |

---

## Crates

| Order | Crate | Group | Must | Should | Fuzz | Total |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | `tunnel-session` | tunnel/ | 52 | 11 | 6 | 69 |

### Notes

Single-crate stage but the highest contract density in the entire
plan (69 contracts, 52 Must-tier). The session domain covers
stateful lifecycle management, bidirectional data motion, migration,
and protocol encoding — all in one behavioral loop.

---

## Parity Gate

**Must-tier contracts (52):**

- `SESSION.v3_*` — datagram wire format encode/decode
- `SESSION.muxer_*` — muxer lifecycle (register, unregister,
  migrate, duplicate handling)
- `SESSION.manager_*` — session manager event loop
- `SESSION.serve_*` — session serve lifecycle (idle timeout,
  migration, context cancellation, read errors)
- `SESSION.pipe_close_*` — bidirectional pipe half-close propagation
- `SESSION.close_idempotent` — idempotent session close
- `SESSION.legacy_close_idle` — legacy v2 idle close

**Fuzz targets (6):** Datagram decode robustness — these run
continuously in CI from this stage onward.

All 52 Must contracts must be `green` before stage 3.5 begins.

---

## Key Deliverables

- `Session` with read/write task split, idle timer reset on activity,
  idempotent close
- `SessionManager` — v2 single-task event loop, v3 concurrent with
  locking
- Datagram muxer (`DatagramConn.Serve` errgroup: session manager +
  datagram receive + ICMP router)
- Session migration state machine (context rebinding — old context
  cancellation does not kill migrated session)
- Rate-limited registration, duplicate registration handling
- `PipeBidirectional` stream pipe (half-close propagation, panic
  recovery)
- Packet encode/decode
- ICMP packet router (Linux `SOCK_DGRAM` via `common-sys::network`)

---

## Invariants Active

| ID | Invariant | Enforcement |
| --- | --- | --- |
| SESSION-1 | Idle timer — 5 min default, reset on activity | `SESSION.serve_idle_timeout` |
| SESSION-2 | Close is idempotent | `SESSION.close_idempotent` |
| SESSION-3 | Migration rebinds context without killing session | `SESSION.serve_migrate` |
| SESSION-4 | Duplicate registration resets timer, returns `ResponseOk` | `SESSION.muxer_register_twice` |
| PROXY-4 | Half-close propagation in `PipeBidirectional` | `SESSION.pipe_close_*` |
| ERR-1 | No panics on malformed datagram input | Fuzz targets |

---

## Risks

| Risk | Description | Mitigation |
| --- | --- | --- |
| R2.2 | Session migration stale cancellation bindings | ADR-016 lifecycle contract, explicit token rebinding |
| R2.3 | Channel capacity mismatches change close behavior | Track semantic buffer sizes per channel |
| R3.4 | Session close errors lose severity metadata | Preserve severity-bearing error types |

---

## Session Guidance

- **Session count:** 1–2 sessions (single crate, but high density)
- **Cognitive load:** High — stateful session lifecycle, migration
  state machine, concurrent v3 session manager, ICMP routing
- **Critical path:** On the critical path (3.3 → **3.4** → 3.5)
- **Fuzz:** Set up `cargo fuzz` targets for datagram decode in this
  stage; they run continuously from here onward
- **Cargo check:** `cargo check --workspace` after completion
