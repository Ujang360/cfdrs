# Phase 3.3 — Transport

| | |
| --- | --- |
| Stage | 3.3 |
| Scope | QUIC transport, edge discovery, TLS, connection lifecycle |
| Entry condition | Stage 3.0 exit gate passed AND stage 3.2 exit gate passed |
| Exit condition | All 30 Must-tier contracts green |
| Context risk | High (~112K practical) — split into 2 sub-stages; Extended agent recommended |

---

## Crates

| Order | Crate | Group | Must | Should | Skip | Total |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | `common-sys` | common/ | 0 | 0 | 0 | 0 |
| 2 | `tunnel-transport` | tunnel/ | 13 | 3 | 6 | 22 |
| 3 | `tunnel-connection` | tunnel/ | 17 | 5 | 0 | 22 |

### Implementation order

`common-sys` first — provides safe wrappers over raw sockets,
signals, and file operations. No parity contracts of its own but
consumed by `tunnel-transport`.

`tunnel-transport` second — QUIC connection establishment, edge
discovery, TLS configuration, worker group thread construction.

`tunnel-connection` third — depends on `tunnel-transport` and
`tunnel-rpc`. Single connection lifecycle from registration through
graceful unregister.

### Intra-stage dependency chain

```text
common-sys
  ↑
tunnel-transport  (also imports tunnel-core, common-wire-primitives,
                   common-observability)
  ↑
tunnel-connection (also imports tunnel-rpc, common-retry,
                   common-signal, common-observability)
```

---

## Parity Gate

**Must-tier contracts (30):**

`tunnel-transport` (13):

- `TRANS.quic_server_*` — 4 proxy type dispatch contracts
- `TRANS.edge_*` — 8 edge discovery contracts
- `TRANS.addr_used_by` — address tracking

`tunnel-connection` (17):

- `TRANS.control_stream_registration` — control stream lifecycle
- `TRANS.protocol_selector_new` — protocol selection
- `TRANS.protocol_auto_refresh` — protocol auto-refresh
- 14 additional connection lifecycle contracts

**Skipped (6):** HTTP/2 transport contracts per
[ADR-001](../../adr/001-quic-transport.md) — QUIC-only in RR.

All 30 Must contracts must be `green` before stages 3.4, 3.5, or 3.6
begin.

---

## Key Deliverables

- Safe OS primitive wrappers in `common-sys` (raw sockets, signals,
  file descriptor locking)
- tokio-quiche QUIC connection with zero-copy sends
- Edge address discovery (DNS SRV + cfapi fallback), address pool
- BoringSSL TLS config with PQ curve priority
  (`X25519MLKEM768:X25519Kyber768Draft00:X25519`)
- Worker group thread construction (transport: 4 threads, proxy:
  remaining cores)
- Bounded MPMC channel (`crossbeam-channel`) for `SessionAssignment`
- Registration state machine (Connecting → Registered → Unregistering
  → Stopped)
- `serveControlStream` loop, `UpdateConfiguration` dispatch
- Connection observer event emission

---

## Invariants Active

| ID | Invariant | Enforcement |
| --- | --- | --- |
| ARCH-2 | `quiche::Connection` `!Send` — pinned to birth thread | Compile-time + `tunnel-transport::workers` |
| ARCH-3 | Worker group boundary errors are `common-error` types | Type system |
| ARCH-6 | Every async entry point carries cancellation token | `common-signal` integration |
| TRANS-1 | QUIC connection serve loop handles all proxy types | `TRANS.quic_server_*` contracts |
| TRANS-2 | Protocol selection parity | `TRANS.protocol_selector_new` |
| TRANS-3 | Edge exhaustion + fallback behavior | `TRANS.edge_no_addrs_left` |

---

## Risks

| Risk | Description | Mitigation |
| --- | --- | --- |
| R2.1 | Unbounded stream/task fan-out under QUIC | Bounded concurrency guard in transport accept loop |
| R2.6 | `quiche::Connection` `!Send` thread-pinning | ADR-018 worker group ownership enforcement |
| R5.1 | QUIC wire format must match Go exactly | Byte-level round-trip contracts |
| R1.5 | `defer`-based cleanup differs from Go | Explicit shutdown and drop semantics |

---

## Session Guidance

- **Session count:** 3 sessions (one per crate, strictly sequential)
- **Cognitive load:** High — QUIC transport is the most complex
  behavioral domain; `!Send` constraints require careful thread
  architecture; BoringSSL FFI integration
- **Critical path:** This stage is on the critical path — delays
  here delay stages 3.4, 3.5, 3.6, and transitively 3.8/3.9
- **Cargo check:** `cargo check --workspace` after each crate
