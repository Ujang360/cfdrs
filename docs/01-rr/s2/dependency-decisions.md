# S2.1 — Dependency Decisions

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 — SUBSTRATE |
| S2.1 status | Closed |
| Rust edition | 2024 (MSRV 1.85+) |

This document records architectural dependency *capability* decisions
for the [cfdrs](https://github.com/Ujang360/cfdrs) rewrite. Crate names are not finalized here — they are
decided in S2.6 when crate boundaries are drawn, or at first use in S3.

**Maintenance:** This document remains open for post-2.1 additions.
Any addition must satisfy the addition policy below.

---

## Companion: Shopping Cart

The selection rationale, scoring methodology, alternative candidates,
and architectural decision framework that informed these decisions are
in the [dependency shopping cart](dependency-shopping-cart.md).

| Document | Purpose |
| --- | --- |
| [Shopping cart](dependency-shopping-cart.md) | Research — all candidates, T/M/S scores, alternatives, risk analysis |
| This document | Decisions — selected capabilities, architectural constraints, open ADRs |

---

## Addition Policy

Any dependency added post-2.1 must:

1. Pass T/M/S scoring (Trustworthiness / Maturity / Security, 1–5 each)
   using the same rubric as the [shopping cart](dependency-shopping-cart.md)
2. Document which S1 catalog(s) it serves (behavioral evidence required)
3. Explain why no existing decided capability covers the need

---

## Workspace Configuration

```toml
[workspace]
edition = "2024"
resolver = "2"

[workspace.features]
# Mutually exclusive — compile_error! enforced in crate roots
logging-native  = []   # tracing-native structured spans
logging-compat  = []   # zerolog-compatible JSON output

cli-native      = []   # idiomatic clap-style derive
cli-compat      = []   # urfave/cli cadence match (S4 if parity gap found)

# Capability flags — feature details deferred, future phases may change
more-metrics    = []   # extra QUIC listener + tokio task metrics
alloc-mimalloc  = []   # high-performance allocator
fips            = []   # FIPS build — deferred, not S3 scope
```

Mutually exclusive enforcement pattern (in each crate root):

```rust
#[cfg(all(feature = "logging-native", feature = "logging-compat"))]
compile_error!("logging-native and logging-compat are mutually exclusive");

#[cfg(not(any(feature = "logging-native", feature = "logging-compat")))]
compile_error!("exactly one of logging-native or logging-compat must be enabled");
```

Same pattern applies to `cli-native` / `cli-compat`.

---

## Layer 1 — Runtime and Concurrency

### 1.1 Async Runtime ✅

**[tokio](https://crates.io/crates/tokio)** — selected. `new_multi_thread()`, all threads pinned via
`sched_setaffinity` in `on_thread_start`. No `LocalSet`. No
`new_current_thread` runtimes. Single multi-thread runtime for both
Tunnel EVL and Proxy EVL threads. Everything `Send`.

**[glommio](https://crates.io/crates/glommio)** — OUT permanently. QUIC transport incompatibility,
ecosystem lockout.

**io_uring** — OUT for now. Kernel version floor concern. Revisit
post-S4 if benchmarks warrant.

### 1.2 Async Utilities ✅

| Capability | Decision |
| --- | --- |
| Cancellation / context propagation | CancellationToken — maps 1:1 to Go's `context.WithCancel` and `graceShutdownC` broadcast |
| Stream combinators | [tokio](https://crates.io/crates/tokio) stream utilities |
| Async trait extensions, try_join | [futures](https://crates.io/crates/futures) 0.3.x ecosystem |
| Async fn in traits | Rust 2024 native — no proc macro needed |
| `dyn Trait` async | [async-trait](https://crates.io/crates/async-trait) pattern — only where `dyn Trait` is unavoidable |
| Custom futures / pinning | [pin-project-lite](https://crates.io/crates/pin-project-lite) pattern — zero dep preferred |

### 1.3 Channels ✅

**MPMC channel** capability decided for all EVL and Actor boundaries.
Go-style select semantics, production-proven. Tunnel EVL → Proxy EVL
`SessionAssignment` handoff uses a bounded channel of this type.

**[tokio](https://crates.io/crates/tokio) channels** — used internally within async tasks (watch,
oneshot, mpsc). Not for cross-EVL boundaries.

Single-producer single-consumer alternatives — OUT. MPMC covers all
cases.

### 1.4 Synchronization ✅

| Capability | Decision |
| --- | --- |
| Concurrent session registries | Concurrent hashmap capability (sharded, lock-free reads) |
| Hot-path config atomic swap | Atomic Arc-swap capability |
| One-time init (7 sites from S1) | `std::sync::OnceLock` — stdlib, no extra dep |
| Fast mutex / rwlock | Faster-than-std mutex capability (no poisoning) |
| Shared ownership, no weak refs | Lean Arc capability — no weak ref overhead, saves 8 bytes per allocation on session handles and config snapshots |
| Per-thread bump arena | Bump arena capability — `!Sync`, reset() at session boundary. Only temporaries go in. Anything that outlives the session goes to heap. Arena discipline is an architectural invariant. |

**Slot-based allocators (slab, etc.)** — OUT. Concurrent hashmap
covers session registries.

### 1.5 Signal Handling ✅

**Runtime-agnostic signal handling** — decided. Works outside async
context. SIGINT / SIGTERM for graceful shutdown. Go has 3
signal-related goroutines; this maps cleanly.

### 1.6 Memory Allocator ✅

**High-performance allocator** capability decided, behind
`alloc-mimalloc` feature flag.

Known required features: `extended`, `local_dynamic_tls`, `no_thp`,
`override`. Benchmark in S4.

**Conflict check:** The QUIC transport dependency tree may transitively
pull in another allocator as an opt-in feature. That feature must not
force `#[global_allocator]` on the binary — the [cfdrs](https://github.com/Ujang360/cfdrs) allocator
declaration wins. **Verify on first build in S3.**

If link conflict is unresolvable at build time: fallback to pure-Rust
QUIC transport path with a FIPS-capable TLS backend. The fallback path
is pre-identified and viable.

### 1.7 Rate Limiting ✅

**GCRA rate limiter** capability — [tokio](https://crates.io/crates/tokio)-aware. Replaces `flow/limiter`
(~200 Go lines). Token-bucket / GCRA algorithm, 64-bit atomic state.

### 1.8 Caching ✅

**Async TinyLFU cache** capability — TTL, TTI, size-bounded eviction.
Replaces ad-hoc `sync.Map` caching for feature flags and edge address
pool.

---

## Layer 2 — Serialization and Data

### 2.1 Core Serialization ✅

| Capability | Decision |
| --- | --- |
| General serialization | [serde](https://crates.io/crates/serde) ecosystem |
| JSON | [serde_json](https://crates.io/crates/serde_json) |
| YAML | [serde_yaml](https://crates.io/crates/serde_yaml) 0.9.x — deprecated but stable. Migrate when maintained fork reaches 0.1.x+ maturity. |
| TOML | [toml](https://crates.io/crates/toml) — parity TOML contracts, workspace config |
| Cap'n Proto runtime | [capnp](https://crates.io/crates/capnp) **0.25.x** — updated from [shopping cart](dependency-shopping-cart.md) 0.20.x |
| Cap'n Proto codegen | [capnpc](https://crates.io/crates/capnpc) **0.25.x** — build-time |
| Cap'n Proto RPC | [capnp-rpc](https://crates.io/crates/capnp-rpc) **0.25.x** — self-proxy service, same-process, actor-connected |
| Protobuf | [prost](https://crates.io/crates/prost) — datagram v2 tracing spans (frame type `0x03`). Passively maintained; migrate when official protobuf crate matures. |

**[capnp-rpc](https://crates.io/crates/capnp-rpc)** is treated as a self-proxy service variant — session-based,
dispatched through the same proxy invariant as all other tunnel traffic.
Never granted architectural special status. Never classified as
transport control.

### 2.2 Data Primitives ✅

Zero-copy byte buffer, UUID, URL, HTTP vocabulary types (request,
response, header map, status code), HTTP body traits, MIME types, percent
encoding — all IN. See [shopping cart](dependency-shopping-cart.md) for
versions and catalog spans.

### 2.3 Encoding ✅

Base64, hex encoding — IN.

### 2.4 Stack-Allocated Data Structures ✅

| Capability | Decision | Rationale |
| --- | --- | --- |
| Fixed-capacity stack collections | [arrayvec](https://crates.io/crates/arrayvec) — `ArrayString<N>` for known-max-length identifiers (ALPN, protocol names, metric labels). `ArrayVec<T, N>` for fixed-capacity hot-path collections. | Compile-time capacity, Copy-able |
| Small-string optimization (dynamic) | [compact_str](https://crates.io/crates/compact_str) pattern — SSO for dynamically-sized strings: config keys, header values, ingress rule hostnames | Inline ≤24 bytes, transparent heap fallback |
| Small vec (heap fallback) | [smallvec](https://crates.io/crates/smallvec) — header lists, datagram frame metadata, small hot-path collections | Inline N elements, heap fallback |
| Byte-string operations | [bstr](https://crates.io/crates/bstr) — 150+ byte/string conversion sites from S1 porting-friction catalog | Go's implicit `[]byte` ↔ `string` |
| Immutable SSO strings | OUT — [arrayvec](https://crates.io/crates/arrayvec) + [compact_str](https://crates.io/crates/compact_str) cover the space | |
| Fixed-size ASCII identifiers | OUT — [arrayvec](https://crates.io/crates/arrayvec) ArrayString covers this | |
| no_std fixed collections | OUT — Linux-only target, no no_std constraint | |

**Stack overflow awareness:** TypeState machines, fat enum payloads,
bump arena temporaries, and stack-allocated strings may accumulate on
deep async call stacks. Rule: `Box` large enum variant *payloads* —
not the enum itself. Audit composite struct sizes with stdlib const
assertions before S3. This applies everywhere, not just enums.

### 2.5 Zero-Copy Data Casting ✅

**Safe zero-copy casting** capability — compile-time enforced. Handles
endianness via typed field annotations on wire structs (`BigEndian`,
`NetworkEndian`). Supersedes both transmute-based casting and separate
endian-read utilities.

x86-64 is little-endian. Wire protocols (QUIC, capnp, network) are
big-endian. The casting capability handles this at struct definition
level, zero-cost.

### 2.6 Parsing ✅

**Combinator-based binary parser** — IN for custom binary wire formats:
datagram v2/v3 frame types (0x00–0x03), QUIC stream 6-byte preamble,
Cap'n Proto frame boundaries.

Decision recorded in [ADR-004](adr/004-parser-strategy-combinator-vs-manual.md).

---

## Layer 3 — Transport

### 3.1 QUIC ✅

**Cloudflare's [tokio](https://crates.io/crates/tokio)-integrated QUIC** capability — selected.

Feature set decisions:

- Zero-copy sends: **mandatory**
- Google congestion control (gcongestion): **mandatory** (implied by zero-copy)
- Extra QUIC listener metrics + [tokio](https://crates.io/crates/tokio) task metrics: behind `more-metrics` flag

The QUIC transport has **two dispatch paths on one connection:**

- Stream-based proxy (HTTP, WebSocket, TCP via ConnectRequest)
- Datagram-based proxy (UDP v2/v3, ICMP)

Both coexist on the same connection. Expressed as one connection type
with two dispatch paths in the tunnel transport abstraction, not two
separate implementations.

**Pure-Rust QUIC** — conditional fallback only if allocator link
conflict is unresolvable. Not the active path.

**0-RTT** — deferred post-S4.

### 3.2 HTTP/2 Transport ✅

HTTP/2 tunnel implementation — deferred. Abstraction layer required
now.

**`cfdrs-tunnel-transport`** — crate name pre-decided for the
abstraction seam between transport and proxy (exception to the S2.6
naming rule: critical architectural seam). Trait surface must
accommodate both QUIC and HTTP/2 as future implementations without
assuming QUIC-only. Internal structure (traits, types) decided in S2.6.

### 3.3 TLS ✅

BoringSSL (baked into QUIC transport dependency) — mandatory on the
primary path. No separate TLS crate needed on this path.

**FIPS:** `fips = []` feature flag in workspace. Not S3 scope.
BoringSSL build flag only, no new crates at this phase.

### 3.4 HTTP Client ✅

Async HTTP client capability — IN. Evaluate latest major version
([shopping cart](dependency-shopping-cart.md) lists 0.12.x; 0.13.x is
available). For cfapi REST calls and upstream API contracts.

### 3.5 WebSocket ✅

[Tokio](https://crates.io/crates/tokio)-integrated WebSocket capability — IN. Management log stream
WebSocket, carrier WebSocket proxy sessions.

### 3.6 Service / Middleware ✅

**Service middleware** — management server only. CORS + timeout layers.
**Never on proxy hot path. Never on session hot path.**

**Management HTTP server** — `/ping`, `/host_details`, `/metrics`,
`/logs` (WebSocket). Not for proxy traffic.

Decision recorded in [ADR-006](adr/006-management-http-stack.md).

---

## Layer 4 — Observability

### 4.1 Logging ✅

Structured async-aware logging — `logging-native` flag.

Bridge for third-party crates using the `log` crate — IN.

Structured journald output — IN for Linux/systemd deployments.

Error type span enrichment (captures active [tracing](https://crates.io/crates/tracing) span stack when
error is created) — IN.

`logging-compat` path: custom subscriber Layer outputting
zerolog-compatible JSON. Custom build regardless of logging flag.

`ConnAwareLogger` equivalent: custom Layer choosing warn/error based on
active connection count (S1: `connection/observer` behavior).

Management WebSocket log stream: custom Layer serializing events to
WebSocket client.

### 4.2 Distributed Tracing ✅

OpenTelemetry ecosystem — IN. Datagram v2 propagates trace identity
through frame type `0x03`. Pre-1.0 but actively developed.

### 4.3 Metrics ✅

**Prometheus-compatible metrics** — direct, no facade. Registers
counters/gauges across connection, quic, supervisor, datagramsession,
flow, ingress, tunnelrpc.

Decision recorded in [ADR-003](adr/003-metrics-facade-vs-direct-prometheus.md).

### 4.4 Error Reporting ✅

Panic capture with backtrace — IN. Mirrors Go's Sentry integration
(panic recovery in bidirectional stream pipe, S1 evidence).

### 4.5 Runtime Debugging ✅

Async runtime task inspector — behind `more-metrics` or separate
`debug-console` feature gate. S4 profiling only. Never in production
builds.

---

## Layer 5 — Error Handling ✅

### Architecture (derived from S1 error-propagation catalog)

The Go codebase has a homegrown `(error, recoverable bool)` dual-return
pattern across `serveTunnel`, `serveConnection`, `serveQUIC`. Five
type-switch classification sites, 15+ error types, layered recovery
semantics where the same error type has different recoverability at
different stack depths.

```text
thiserror 2.x     → typed error enums in all library crates.
                    Recoverability encoded as a method on the enum:
                      fn is_recoverable(&self) -> bool

anyhow            → OUT entirely. Not "binary only." Out.

std::process::ExitCode
                  → fatal errors map here at binary boundary.
                    Each fatal variant gets a typed exit code.

? operator        → NEVER at EVL or Actor boundary.
                    EVLs handle errors internally, send typed
                    actor messages upward. Error never escapes
                    the EVL via propagation.

? operator        → allowed inside pure computation within a task,
                    never crossing the task/EVL boundary.
```

**Go string-match heuristics → typed variants in Rust (S1 quirks
fixed at the rewrite boundary):**

- `strings.Contains(err.Error(), "Unauthorized")` →
  `RegistrationError::Unauthorized`
- `*net.OpError` + `"operation not permitted"` →
  `TransportError::EgressBlocked`

**[miette](https://crates.io/crates/miette)** — 🔲 open. Evaluate for CLI diagnostic display in S4.
**[eyre](https://crates.io/crates/eyre)** — OUT.

---

## Layer 6 — CLI and Configuration ✅

**CLI framework** — `cli-native` flag. Derive-macro-based subcommand
trees.

`cli-compat` — deferred. Add in S4 only if parity tests reveal
behavioral gap vs urfave/cli cadence.

**Timestamp / duration** — [chrono](https://crates.io/crates/chrono) ecosystem for formatting and parsing.
Human-readable duration parsing for config values (`"5s"`, `"210s"`,
`"1h"`).

Key timeouts from S1: idle 210s, RPC 5s, dial edge 15s, heartbeat 15s,
management idle 5min, feature refresh 1h, registration interval 1s.

**Platform paths** — XDG-compliant directory discovery. `.env` file
loading for development.

Config search path (S1 evidence): `~/.cloudflared` →
`/etc/cloudflared` → `$CFDPATH` → systemd unit env.

---

## Layer 7 — Domain-Specific

### Actor Framework ✅

**Erlang-inspired typed actor framework** — selected. Supervision tree,
upward error escalation.

[capnp-rpc](https://crates.io/crates/capnp-rpc) self-proxy service: same OS process, own EVL, communicates
to high-level system actors via actor message passing. Treated as a
special proxy variant — session-based, dispatched through the same
proxy invariant. Never granted architectural special status.

Session managers: custom for session-based EVL. Actor framework for
supervisor orchestration and actor address-book graph.

### Backoff / Retry ✅

**Async-compatible retry with backoff** — IN.

Decision recorded in [ADR-005](adr/005-retry-backoff-library-vs-custom.md).

### SOCKS5 ✅ — ADR-007 Resolved

**Custom implementation. Build, not Buy.**

cloudflared implements a SOCKS5 **server** that processes proxied tunnel
traffic. The SOCKS5 client resides on the remote/edge side. There is no
outward SOCKS5 proxying. The full flow:

```text
Remote SOCKS5 client
  → Cloudflare Edge
  → tunnel (WebSocket carrier)
  → cloudflared SOCKS5 server
  → dialer
  → TCP origin
```

SOCKS5 runs over the WebSocket carrier — not raw TCP. The WebSocket
stream is the transport; cloudflared speaks the SOCKS5 server protocol
over it.

S1 evidence (5 atoms, ~400 lines, zero external deps):

- `socks/auth_handler` — method-map auth negotiation
- `socks/authenticator` — NoAuth + UserPassAuth (RFC 1929) only
- `socks/connection_handler` — sequential auth → request pipeline
- `socks/dialer` — NetDialer + ConnDialer, dials TCP origin
- `socks/request_handler` — CONNECT only (BIND + ASSOCIATE are stubs),
  IP access policy enforcement, stream piping to origin

A library would provide more than needed and lacks awareness of the
WebSocket carrier and IP access policy integration. Custom wins cleanly.

Classification: **sessionless proxy data** — dispatched from the fat
enum at ingress, per-request handler, no session registry.

### Other Domain Capabilities ✅

Async DNS resolver, file watcher (config hot-reload), IP/CIDR matching,
regex (ingress rule path matching), gzip compression, diagnostic zip
bundling, FNV hashing (feature flag percentile rollout) — all IN.

**No autoupdate** in [cfdrs](https://github.com/Ujang360/cfdrs).

**Foundations** (Cloudflare bootstrap crate) — OUT as direct
dependency. Accepted as transitive dep from QUIC transport (single
telemetry feature only).

---

## Layer 8 — Crypto and Security ✅

All RustCrypto capabilities — IN: SHA-2, SHA-1 (WebSocket handshake),
HMAC, ECDSA, P-256, Ed25519, Curve25519/X25519, AES-GCM, NaCl sealed
box (XSalsa20-Poly1305), random generation, PEM/DER/PKCS8/x509
certificate handling, JWT validation (RS256), SSH key generation.

**secrecy + zeroize** — deferred to crypto crate boundary. Not S3
scope.

**BoringSSL** — transitive via QUIC transport. Not a direct dependency.

---

## Layer 9 — Platform ✅

**Target:** Linux only, x86-64-v2/v4 + systemd. No container for now.
CLI quick run. strip + fat LTO. No NUMA concern. No autoupdate.
No Windows/macOS in this phase.

| Capability | Decision |
| --- | --- |
| Unix syscall wrapper | IN — sched_setaffinity, socket options, ICMP raw sockets, ping-group detection, DF bit |
| Raw FFI bindings | IN — where the safe wrapper does not cover |
| Systemd readiness | **[sd-notify](https://crates.io/crates/sd-notify)** — READY=1 and STOPPING=1 only. Minimal. ADR-009 closed. |
| UDP socket tuning | IN — QUIC UDP socket buffer sizes, DF bit, platform-specific options |
| System info collection | IN — CPU, memory, disk, network for diagnostics |
| File descriptor locking | IN — token file / credential exclusive access |
| Hostname detection | IN |
| Executable discovery | IN — traceroute for diagnostics |
| Browser launch | IN — access token flows (xdg-open on Linux) |

---

## S2.4 ADRs

Decision records produced in phase 2.4:

| ADR | Decision | Origin | Blast Radius |
| --- | --- | --- | --- |
| [ADR-003](adr/003-metrics-facade-vs-direct-prometheus.md) | Prometheus client vs metrics facade | Layer 4.3 | Medium — all instrumented modules |
| [ADR-004](adr/004-parser-strategy-combinator-vs-manual.md) | Combinator parser vs manual byte-buffer slicing | Layer 2.6 | Medium — wire format parsing |
| [ADR-005](adr/005-retry-backoff-library-vs-custom.md) | Async backoff library vs custom implementation | Layer 7 | Low — retry logic |
| [ADR-006](adr/006-management-http-stack.md) | Full web framework vs raw HTTP for management server | Layer 3.6 | Low — management server only |

Expanded decisions tracked in [ADR index](adr/README.md):

| ADR | Decision focus | Origin |
| --- | --- | --- |
| [ADR-018](adr/018-quic-thread-affinity-enforcement.md) | QUIC thread-affinity enforcement | R2.6 / Layer 1.1 and 3.1 |
| [ADR-019](adr/019-cancellation-timeout-propagation-contract.md) | Cancellation and timeout propagation contract | R1.2 |
| [ADR-012](adr/012-error-taxonomy-and-recoverability-policy.md) | Error taxonomy and recoverability policy | R3.1 |
| [ADR-013](adr/013-customduration-dual-format-contract.md) | CustomDuration dual-format serialization contract | R1.6 |
| [ADR-014](adr/014-startup-dag-contract.md) | Startup DAG contract | R4.1 |
| [ADR-015](adr/015-graceful-shutdown-contract.md) | Graceful shutdown contract | R4.2 |
| [ADR-016](adr/016-session-migration-lifecycle.md) | Session migration lifecycle contract | R2.2 and R5.3 |
| [ADR-017](adr/017-access-s4-drop-in-contract.md) | Access S4 drop-in replacement contract | Scope Access reclassification / R6.6 |

### Resolved ADRs

| ADR | Resolution | Rationale |
| --- | --- | --- |
| ADR-007 | SOCKS5 — build, not buy | S1 scope is minimal (~400 lines), WebSocket carrier integration, IP access policy not available in libraries |
| ADR-009 | [sd-notify](https://crates.io/crates/sd-notify) — minimal sd_notify(3) | READY=1 + STOPPING=1 only, no full systemd service lifecycle |

### Tentative Evaluations

| Candidate | Evaluation Window | Purpose |
| --- | --- | --- |
| [miette](https://crates.io/crates/miette) | S4 | CLI diagnostic display enrichment |

---

## S2.1 Exit Gate

| Criterion | Status |
| --- | --- |
| All 9 capability layers documented | ✅ |
| Every decision traceable to S1 catalog evidence | ✅ |
| Workspace feature flag architecture established | ✅ |
| ADR-003/004/005/006 decision records authored in S2.4 | ✅ |
| Resolved ADRs documented with rationale | ✅ ADR-007, ADR-009 |
| Platform target decided | ✅ Linux x86-64, systemd |
| Build-vs-buy evaluated for all domain capabilities | ✅ |

**Phase 2.1 status: Closed.**

---

## S2.2 Entry Conditions

Phase 2.2 (Scoping) consumes the following from this document:

### Platform Matrix Input

- **Target:** Linux x86-64-v2/v4 + systemd (Layer 9)
- **Excluded this phase:** Windows, macOS, container, NUMA, io_uring
- **Deferred:** FIPS build (`fips = []` flag defined, not S3 scope)

### MoSCoW Input

| Priority | Capabilities |
| --- | --- |
| Must | [tokio](https://crates.io/crates/tokio) runtime, QUIC transport, Cap'n Proto RPC, actor framework, structured logging, typed errors |
| Should | MPMC channels, concurrent hashmap, bump arena, TinyLFU cache, GCRA rate limiter |
| Could | logging-compat (zerolog JSON), cli-compat, runtime debugging (debug-console) |
| Won't (this phase) | [glommio](https://crates.io/crates/glommio) runtime, io_uring, FIPS, autoupdate, Windows/macOS, 0-RTT |

### Non-Port Decisions

- Auto-updater — not ported to [cfdrs](https://github.com/Ujang360/cfdrs)
- Foundations crate — OUT as direct dependency
- [glommio](https://crates.io/crates/glommio) — OUT permanently (QUIC transport incompatibility)
- Windows/macOS platform — excluded this phase
- 0-RTT — deferred post-S4
