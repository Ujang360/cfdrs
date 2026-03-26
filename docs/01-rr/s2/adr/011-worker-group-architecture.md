# ADR-011 - Worker Group Architecture

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Decided |
| Origin | [dependency-decisions](../dependency-decisions.md) § 1.1 and Layer 1 |
| Risk linkage | R2.6 |

## Question

What is the precise worker group architecture — thread count,
pinning policy, assignment strategy, and inter-group boundary rules?

## Decision

Two worker groups, both owned by `tunnel-transport::workers`:

| Worker group | Threads | Pinning | Hosts |
| --- | --- | --- | --- |
| Transport worker group | 4 | `sched_setaffinity` via `common-sys::cpu` | QUIC connection serve loops |
| Proxy worker group | Remaining cores | `sched_setaffinity` via `common-sys::cpu` | Session serve loops, proxy dispatch |

Single multi-thread [tokio](https://crates.io/crates/tokio) runtime.
All threads pinned via `sched_setaffinity` in `on_thread_start`.
`quiche::Connection` is `!Send` — pinned to birth thread forever.
No connection migration after assignment.

**Assignment:** `min_by_key` on atomic counter per proxy worker group
thread at assignment time.

**Worker boundary types** (in `tunnel-core`):
`TunnelWorkerHandle` (`!Send`), `ProxyWorkerHandle` (`!Send`),
`SessionAssignment`, `WorkerIndex`.

**MPMC channel at worker boundary:** `SessionAssignment` messages
cross from transport worker group to proxy worker group via bounded
MPMC (`crossbeam-channel`). tokio channels used internally within
tasks only.

Full crate-level formalization in
[architecture](../architecture.md) § ADR-011 Resolution.

## Consequences

- S3.0 foundation work depends on the worker boundary types being
  locked. This ADR is now decided.
- `tunnel-transport::workers` module owns thread construction and
  pinning logic.
- `tunnel-core` owns the typed boundary handles.
- No `LocalSet` or `new_current_thread` runtimes anywhere.
