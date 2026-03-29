# Phase 2.1 — Dependencies Vetting

Comprehensive dependency shopping cart for the cloudflared Rust rewrite.
Every candidate is traced to S1 catalogs and scored on three axes:
trustworthiness, maturity, and security. Multiple alternatives are
listed per domain to support architectural decisions in later phases.

**Baseline:** cloudflare/cloudflared @ tag `2026.3.0`
**Source:** All 30 S1 catalogs (22 domain + 8 cross-cutting)
**Status:** Shopping cart — decisions finalized in [dependency-decisions.md](dependency-decisions.md)

## Methodology

### Scoring Axes (1–5)

| Score | Trustworthiness | Maturity | Security |
| --- | --- | --- | --- |
| 5 | Rust project / major org, >5 maintainers | Stable API, >5y, 1.0+ | Audited, no CVEs, minimal unsafe |
| 4 | Well-known org, >3 maintainers | Stable API, >3y | Community-reviewed, rare CVEs |
| 3 | Active maintainer, known contributor | Usable API, >1y | No known CVEs, moderate unsafe |
| 2 | Single maintainer, active | Pre-1.0, API churn | Unaudited, some unsafe |
| 1 | Unmaintained / unknown | Experimental | Known vulnerabilities or heavy unsafe |

### Dimensions

- **Plumbing** — Low behavioral coupling, foundation crate,
  low-risk substitution
- **Behavioral** — High Jaccard overlap, shapes architecture,
  high-stakes substitution
- **Domain** — Moderate coupling, serves specific catalog domains
- **Platform** — OS/build-target conditional compilation
- **Crypto** — Cryptographic operations, FIPS/PQ surface
- **Testing** — Dev-dependency only, test infrastructure
- **Facade** — Abstraction layer enabling future swap-out or
  feature-flagged alternatives

### Span Notation

Catalogs abbreviated: `TUN` tunnels, `PRX` proxying, `TT` tunnels-transport,
`CAP` capnp-rpc, `EDG` edge-interactions, `UAC` upstream-api-contracts,
`CLI` cli, `CFG` config, `CE` const-and-env, `CRY` crypto, `ING` ingress,
`MET` metrics, `OBS` observabilities, `OVW` overwatch, `PLT` platforms,
`SUP` supervisor, `SES` sessions, `SS` shared-state, `SM` state-machines,
`AP` access-policies, `HI` host-interactions, `DEP` deployments,
`CON` concurrency, `EP` error-propagation, `FEA` features,
`IT` init-teardown, `PS` platform-substrates, `PF` porting-friction,
`WP` wire-protocol, `TST` tests

### Future Feature Flags

The architecture must accommodate feature-flagged deviations:

1. **`logging-compat` vs `logging-native`** — zerolog-compatible JSON
   output vs native [tracing](https://crates.io/crates/tracing) structured spans
2. **`cli-compat` vs `cli-native`** — `urfave/cli` cadence match vs
   idiomatic [clap](https://crates.io/crates/clap) derive
3. **`runtime-tokio` vs `runtime-glommio`** — [tokio](https://crates.io/crates/tokio) multi-threaded vs
   [glommio](https://crates.io/crates/glommio) thread-per-core (undecided)
4. **`quic-quinn` vs `quic-quiche`** — pure-Rust QUIC vs
   Cloudflare's C-backed [quiche](https://crates.io/crates/quiche)
5. **Other deviations** — TBD in S2/2.4 ADRs

These flags affect dependency selection. Where relevant, candidates are
tagged with which flag path they serve.

## Decision Architecture Framework

This shopping cart is also a decision workbench. We classify decisions
by architectural blast radius and by the kind of synchronization model
they force.

### Decision Classes (Broader Categorization)

| Class | Purpose | Typical Decisions | Blast Radius |
| --- | --- | --- | --- |
| Cadence | Runtime and execution style | full async vs full sync vs mixed | Whole workspace |
| Coordination | Synchronization and ownership | shared state vs message passing | Cross-crate |
| Abstraction | Trait/facade depth | where to add heavy abstraction layers | Module + crate boundaries |
| Partitioning | Crate/module cuts | service boundaries and public APIs | Workspace topology |
| Operational | Deployment and runtime ops | systemd/journald/windows service model | Platform-specific |
| Governance | Tooling and guardrails | debtmap, policy checks, CI gates | Development workflow |

### Architectural Cadence Models

| Model | Description | Best Fit Areas | Main Risks | Recommended Crates |
| --- | --- | --- | --- | --- |
| Full Async | Async end-to-end runtime | tunnels, transport, ingress, supervisor | async contagion, trait complexity | [tokio](https://crates.io/crates/tokio), [futures](https://crates.io/crates/futures), [quinn](https://crates.io/crates/quinn)/[quiche](https://crates.io/crates/quiche) |
| Full Sync | Threaded/sync core services | small control utilities, CLI-only workflows | lower throughput, blocking I/O pressure | [crossbeam](https://crates.io/crates/crossbeam), [parking_lot](https://crates.io/crates/parking_lot), [ureq](https://crates.io/crates/ureq) |
| Mixed | Async data plane + sync control plane | cloudflared-like split architecture | boundary complexity | [tokio](https://crates.io/crates/tokio) + [crossbeam](https://crates.io/crates/crossbeam) + [flume](https://crates.io/crates/flume) |
| Actor-Heavy | Message passing + supervised actors | overwatch, session lifecycle, supervisor FSM | framework lock-in, mailbox overhead | [ractor](https://crates.io/crates/ractor), [crossbeam-channel](https://crates.io/crates/crossbeam-channel) |
| Hybrid Runtime | tokio default + targeted glommio lanes | Linux high-throughput hotspots | dual-runtime complexity | [tokio](https://crates.io/crates/tokio) + [glommio](https://crates.io/crates/glommio) + adapters |

### Concurrency and Synchronization Pressure Map

Comprehensive subsystem-to-runtime mapping. Every major subsystem from
S1 is assigned a preferred execution model, synchronization strategy,
and actor suitability rating.

| Subsystem | Preferred Model | Data Sync Pressure | Actor Suitability | Why |
| --- | --- | --- | --- | --- |
| Datagram v3 session manager | actor-heavy | High | **Strong** | per-session state, migration coordination, muxer/demuxer split |
| Datagram v2 session manager | actor or mixed | High | **Strong** | single-goroutine event loop in Go maps directly to actor mailbox |
| Edge connection loops (HA×4) | full async | Medium | Weak | high I/O multiplexing, cancellation tokens, select-loop pattern |
| QUIC transport layer | full async | Medium | Weak | stream/datagram I/O is inherently async |
| HTTP/2 transport layer | full async | Medium | Weak | bidirectional stream relay, [hyper](https://crates.io/crates/hyper) is async-native |
| Supervisor restart FSM | actor-heavy | Medium | **Strong** | explicit lifecycle messaging, restart policies, error escalation |
| Overwatch service registry | actor-heavy | Medium | **Strong** | config-driven service replacement, supervised restart |
| Tunnel orchestration | mixed (async + actor) | Medium | Moderate | config reload triggers re-orchestration across connections |
| Config/reload pipeline | mixed | Low | Moderate | write-seldom/read-often; [arc-swap](https://crates.io/crates/arc-swap) or watch channel |
| Feature flag resolver | full async | Low | Weak | periodic DNS+API fetch, broadcast to consumers |
| Ingress rule engine | full sync (hot path) | Low | Weak | stateless rule evaluation per request; no shared mutable state |
| Proxy request dispatch | full async | Low | Weak | per-request; no cross-request state; [tower](https://crates.io/crates/tower) middleware chain |
| SOCKS5 proxy handler | full async | Low | Weak | per-connection; stateless after handshake |
| ICMP proxy (v4 + v6) | mixed | Medium | Moderate | 2 router goroutines in Go; shared echo-ID tracking table |
| WebSocket carrier relay | full async | Low | Weak | bidirectional pipe; no shared state |
| Management HTTP server | full async | Low | Weak | small request volume; standard HTTP server |
| Management WebSocket (logs) | full async | Low | Weak | single-session streaming; [tracing](https://crates.io/crates/tracing) Layer subscription |
| Metrics collection/export | full sync or mixed | Low | Weak | atomic counters; Prometheus scrape is sync HTTP |
| DNS resolution (edge SRV) | full async | Low | Weak | async DNS queries with caching |
| DNS proxy (proxydns) | full async | Low | Weak | request/response forwarding |
| Access token/JWT validation | full sync | None | Weak | stateless cryptographic verification |
| Credential/cert management | mixed | Low | Weak | file watch + reload; infrequent writes |
| Auto-updater | full sync or mixed | Low | Weak | periodic check; self-replace; platform service restart |
| CLI/bootstrap path | full sync | None | Weak | finite, command-oriented execution before runtime starts |
| Diagnostic collection | full sync | None | Weak | one-shot system info collection; no concurrency needed |
| SSH key generation | full sync | None | Weak | one-shot crypto operation |

### Subsystem Actor Assignment Recommendations

Based on the pressure map above, here are explicit actor/non-actor
assignments per subsystem group:

| Group | Subsystems | Actor Framework? | Rationale |
| --- | --- | --- | --- |
| **Session lifecycle** | datagram v2 mgr, v3 mgr, v3 muxer | Yes — [ractor](https://crates.io/crates/ractor) or custom | Go uses single-goroutine event loops; actor mailbox is 1:1 match; migration and registration are message-driven |
| **Supervision tree** | supervisor FSM, overwatch, tunnel orchestration | Yes — [ractor](https://crates.io/crates/ractor) or custom | restart policies, error escalation, graceful shutdown ordering need supervision semantics |
| **Transport I/O** | QUIC, HTTP/2, edge connections, WebSocket relay | No — [tokio](https://crates.io/crates/tokio) tasks + channels | high-throughput I/O; actor overhead (mailbox serialization) hurts; use `CancellationToken` for lifecycle |
| **Request dispatch** | proxy, ingress, SOCKS5, tower middleware | No — [tokio](https://crates.io/crates/tokio) tasks | stateless per-request; tower `Service` trait provides composition without actor overhead |
| **Platform services** | ICMP routers, config reload, auto-updater | Maybe — mixed | ICMP needs shared tracker table (actor-friendly); config reload is simple watch channel; auto-updater is sync |
| **Observability** | metrics, [tracing](https://crates.io/crates/tracing), management server | No — standard async | low contention; atomic counters; standard HTTP server patterns |
| **CLI/oneshot** | CLI, diagnostics, SSH keygen, credential ops | No — sync | no concurrency needed; runs to completion |

### glommio Suitability per Subsystem

If the `runtime-glommio` flag path is pursued, only specific
subsystems benefit from thread-per-core I/O:

| Subsystem | glommio Benefit | Feasibility | Notes |
| --- | --- | --- | --- |
| QUIC datagram relay | **High** — eliminates cross-thread sync | Medium | requires io_uring QUIC adapter; no existing [quinn](https://crates.io/crates/quinn)/[quiche](https://crates.io/crates/quiche) [glommio](https://crates.io/crates/glommio) integration |
| HTTP/2 stream relay | **High** — bidirectional copy is I/O-bound | Low | [hyper](https://crates.io/crates/hyper) assumes [tokio](https://crates.io/crates/tokio); would need custom HTTP/2 layer |
| Session manager | Medium — reduces lock contention | Medium | actor mailbox can be thread-local |
| Supervisor/overwatch | None | N/A | not I/O-bound |
| Management server | None | N/A | low traffic |
| CLI/diagnostics | None | N/A | not I/O-bound |

**Recommendation:** [glommio](https://crates.io/crates/glommio) is viable only for the datagram relay hot
path on Linux. The integration cost is high because the entire QUIC
and HTTP ecosystem assumes [tokio](https://crates.io/crates/tokio). Evaluate only if S4 benchmarks
reveal cross-thread contention as a real bottleneck.

### Abstraction Intensity Classes

| Intensity | When to Use | Examples |
| --- | --- | --- |
| Heavy | backend choice may change and touches many crates | transport facade, runtime facade, logging facade |
| Moderate | external crates vary but API shape is stable | metrics facade, DNS resolver trait |
| Light | crate is stable and low-risk | [serde](https://crates.io/crates/serde) stack, [uuid](https://crates.io/crates/uuid), [percent-encoding](https://crates.io/crates/percent-encoding) |
| Avoid | low-value wrappers that hide useful semantics | generic wrappers around [bytes](https://crates.io/crates/bytes)/[http](https://crates.io/crates/http) types |

### Crate/Module Partitioning Heuristics

| Heuristic | Guidance | Impact |
| --- | --- | --- |
| Ownership First | one owner per mutable state region | reduces lock contention and races |
| API Skinny Core | small cross-crate trait surface | reduces cognitive load and review cost |
| Async Containment | isolate async-heavy APIs by crate | avoids async spread into all modules |
| Platform Edges | isolate `#[cfg]` logic behind adapters | keeps core logic portable |
| Policy in Tooling | move policy checks to guardrails | avoids runtime complexity |

### Cognitive Model and Context-Window Risk

| Risk | Trigger | Architectural Impact | Mitigation |
| --- | --- | --- | --- |
| Decision Scatter | too many ad-hoc choices per crate | inconsistent cadence across modules | decision-class tables + ADR templates |
| Abstraction Sprawl | facade overuse everywhere | slower delivery, hidden behavior | use abstraction intensity classes |
| Runtime Bifurcation | [tokio](https://crates.io/crates/tokio) + [glommio](https://crates.io/crates/glommio) in same paths | debugging complexity | strict boundary crates and adapters |
| Synchronization Drift | mixed lock/message patterns | deadlocks and race windows | ownership-first partitioning |
| Interface Explosion | too many public traits | higher review/context burden | API skinny core rule |

### Architecture Impact Matrix

| Decision Axis | Crate Layout Impact | Concurrency Impact | Testing Impact | Ops Impact |
| --- | --- | --- | --- | --- |
| Runtime choice | core runtime crate + adapters | task scheduling and cancellation model | async vs sync test harness split | signal/service lifecycle handling |
| Transport backend | dedicated transport facade crate | stream/datagram API shape | parity harness matrix expands | TLS/FIPS deployment matrix |
| Actor adoption | actor boundary crates | message-driven ownership model | mailbox and supervision tests | restart/observability behavior |
| Logging mode | logging facade crate | minimal | snapshot/log contract tests | journald/JSON pipeline choices |
| Platform model | platform adapter crates | none | per-OS integration tests | systemd/launchd/windows service paths |

### Data Synchronization Reduction Strategies

Concrete strategies for reducing shared mutable state and lock
contention, organized by the synchronization primitive they replace.

#### Strategy 1: Ownership Transfer (Eliminate Sharing)

| Go Pattern | Rust Replacement | Applicable Subsystems |
| --- | --- | --- |
| `sync.Mutex`-protected map + goroutine reads | Move map ownership into single actor; queries via message | session manager, overwatch registry |
| Shared `*TunnelConfig` pointer | [arc-swap](https://crates.io/crates/arc-swap) for atomic pointer swap; readers never block | config/reload pipeline, orchestration |
| `sync.WaitGroup` for shutdown | `CancellationToken` tree (parent→child hierarchy) | supervisor, connection loops, init-teardown |
| Channel of channels (response pattern) | `tokio::oneshot` per request | RPC request/response, session registration |

#### Strategy 2: Partitioned State (Reduce Contention)

| Go Pattern | Rust Replacement | Applicable Subsystems |
| --- | --- | --- |
| Single `sync.RWMutex` map for sessions | [dashmap](https://crates.io/crates/dashmap) (sharded concurrent map) | session registry (v2/v3) |
| Global metrics registry with mutex | [prometheus-client](https://crates.io/crates/prometheus-client) atomic counters (lock-free) | all instrumented modules |
| Shared logger with level check | [tracing](https://crates.io/crates/tracing) subscriber filtering (lock-free) | all modules |

#### Strategy 3: Event-Driven (Replace Polling)

| Go Pattern | Rust Replacement | Applicable Subsystems |
| --- | --- | --- |
| `select {}` with multiple channels | `tokio::select!` macro | supervisor, connection loops |
| Periodic timer + check | `tokio::time::interval` + `select!` | feature refresh, heartbeat, auto-update |
| `fsnotify` watch + channel | [notify](https://crates.io/crates/notify) crate + `tokio::sync::mpsc` | config reload, credential refresh |

#### Strategy 4: Immutable Broadcast (Write-Once, Read-Many)

| Go Pattern | Rust Replacement | Applicable Subsystems |
| --- | --- | --- |
| Atomic bool flags | `AtomicBool` / `tokio::sync::watch` | graceful shutdown flags, feature toggles |
| `sync.Once` + global var | `std::sync::OnceLock` / `once_cell::Lazy` | 7 init sites from S1 |
| Config snapshot broadcast | `tokio::sync::watch` channel | config hot-reload to all consumers |

### Dependency Interaction and Conflict Map

Dependencies do not exist in isolation. This map documents known
interactions, mutual exclusions, and version-coupling constraints.

#### Mutual Exclusion Pairs

| Crate A | Crate B | Conflict Type | Resolution |
| --- | --- | --- | --- |
| [rustls](https://crates.io/crates/rustls) | [boring](https://crates.io/crates/boring) | TLS backend — cannot both be active for same connection | Cargo feature flags: `tls-rustls` vs `tls-boring` |
| [quinn](https://crates.io/crates/quinn) | [quiche](https://crates.io/crates/quiche) | QUIC backend — different API shapes, different TLS coupling | Cargo feature flags: `quic-quinn` vs `quic-quiche`; transport facade trait |
| [tokio](https://crates.io/crates/tokio) | [glommio](https://crates.io/crates/glommio) | Runtime — incompatible executors (work-stealing vs thread-per-core) | Feature flags: `runtime-tokio` vs `runtime-glommio`; strict crate boundary |
| [serde_yaml](https://crates.io/crates/serde_yaml) | [serde_yml](https://crates.io/crates/serde_yml) | YAML backend — both parse YAML; serde_yaml deprecated | ADR: pick one; serde_yml is maintained fork |
| [chrono](https://crates.io/crates/chrono) | [time](https://crates.io/crates/time) | Time library — overlapping functionality | ADR: pick one per crate, isolate behind time utility module |

#### Tight Coupling Chains

These crates form dependency chains where version bumps cascade:

| Chain | Members | Coupling Risk |
| --- | --- | --- |
| [hyper](https://crates.io/crates/hyper) ecosystem | [hyper](https://crates.io/crates/hyper) 1.x → [http](https://crates.io/crates/http) 1.x → [http-body](https://crates.io/crates/http-body) 1.x → [http-body-util](https://crates.io/crates/http-body-util) 0.1.x | [hyper](https://crates.io/crates/hyper) major version bump forces all to update |
| [tokio](https://crates.io/crates/tokio) ecosystem | [tokio](https://crates.io/crates/tokio) 1.x → [tokio-util](https://crates.io/crates/tokio-util), `tokio-stream`, [tokio-rustls](https://crates.io/crates/tokio-rustls), [tokio-tungstenite](https://crates.io/crates/tokio-tungstenite) | [tokio](https://crates.io/crates/tokio) major bump is workspace-wide event |
| [quinn](https://crates.io/crates/quinn) + [rustls](https://crates.io/crates/rustls) | [quinn](https://crates.io/crates/quinn) 0.11.x → [rustls](https://crates.io/crates/rustls) 0.23.x → [ring](https://crates.io/crates/ring) 0.17.x | [quinn](https://crates.io/crates/quinn) version pins [rustls](https://crates.io/crates/rustls) version |
| [quiche](https://crates.io/crates/quiche) + [boring](https://crates.io/crates/boring) | [quiche](https://crates.io/crates/quiche) 0.22.x → `boring-sys` 4.x → BoringSSL C build | [quiche](https://crates.io/crates/quiche) version pins BoringSSL revision |
| [tower](https://crates.io/crates/tower) stack | [tower](https://crates.io/crates/tower) 0.5.x → `tower-layer` 0.3.x → [tower-http](https://crates.io/crates/tower-http) 0.6.x | [tower](https://crates.io/crates/tower) trait version shared across all middleware |
| [h3](https://crates.io/crates/h3) + [quinn](https://crates.io/crates/quinn) | `h3` 0.0.x → `h3-quinn` 0.0.x → [quinn](https://crates.io/crates/quinn) 0.11.x | [h3](https://crates.io/crates/h3) pre-1.0 moves fast; [quinn](https://crates.io/crates/quinn) pins |
| RustCrypto | [sha2](https://crates.io/crates/sha2), [hmac](https://crates.io/crates/hmac), `digest`, [p256](https://crates.io/crates/p256), [ecdsa](https://crates.io/crates/ecdsa) share `digest` 0.10.x trait | digest trait version must be consistent |
| Cap'n Proto | [capnp](https://crates.io/crates/capnp) 0.20.x → [capnpc](https://crates.io/crates/capnpc) 0.20.x → [capnp-rpc](https://crates.io/crates/capnp-rpc) 0.20.x | single maintainer; versions move in lockstep |
| opentelemetry | [opentelemetry](https://crates.io/crates/opentelemetry) 0.29.x → `opentelemetry_sdk` → `opentelemetry-otlp` → [tracing-opentelemetry](https://crates.io/crates/tracing-opentelemetry) | pre-1.0 API churn; all versions must match |

#### Complementary Pairs (Work Well Together)

| Pair | Synergy | Notes |
| --- | --- | --- |
| [tracing](https://crates.io/crates/tracing) + [tokio](https://crates.io/crates/tokio) | native async span propagation | [tokio](https://crates.io/crates/tokio) instruments tasks with [tracing](https://crates.io/crates/tracing) automatically |
| [hyper](https://crates.io/crates/hyper) + [tower](https://crates.io/crates/tower) | Service trait composition | [hyper](https://crates.io/crates/hyper) 1.x is designed around tower::Service |
| [axum](https://crates.io/crates/axum) + [tower](https://crates.io/crates/tower) + [hyper](https://crates.io/crates/hyper) | full HTTP server stack | [axum](https://crates.io/crates/axum) is built on both |
| [reqwest](https://crates.io/crates/reqwest) + [rustls](https://crates.io/crates/rustls)/[boring](https://crates.io/crates/boring) | TLS backend selection | [reqwest](https://crates.io/crates/reqwest) supports both via features |
| [proptest](https://crates.io/crates/proptest) + [bytes](https://crates.io/crates/bytes) | wire format fuzzing | [proptest](https://crates.io/crates/proptest) strategies generate arbitrary byte sequences |
| [dashmap](https://crates.io/crates/dashmap) + [parking_lot](https://crates.io/crates/parking_lot) | concurrent data structures | [dashmap](https://crates.io/crates/dashmap) uses [parking_lot](https://crates.io/crates/parking_lot) internally |
| [clap](https://crates.io/crates/clap) + [serde](https://crates.io/crates/serde) | config/CLI merge | [clap](https://crates.io/crates/clap) derive + [serde](https://crates.io/crates/serde) derive on same structs |

#### Version Pinning Strategy

| Risk Level | Strategy | Example |
| --- | --- | --- |
| Critical (transport/TLS) | Pin exact minor version in workspace Cargo.toml | `quinn = "=0.11.6"` |
| High (ecosystem chains) | Pin minor range | `hyper = "1.5"` |
| Medium (domain crates) | Pin major only | `serde = "1"` |
| Low (testing/tooling) | Floating within major | `proptest = "1"` |

### S3 Phase-Gate Dependency Schedule

Which dependencies to introduce at each S3 stage. Dependencies are
only added when their stage becomes active, preventing premature
coupling.

| S3 Stage | Dependencies Introduced | Entry Condition |
| --- | --- | --- |
| **3.0 Foundation** | [thiserror](https://crates.io/crates/thiserror), [serde](https://crates.io/crates/serde), [serde_json](https://crates.io/crates/serde_json), [bytes](https://crates.io/crates/bytes), [uuid](https://crates.io/crates/uuid), [tracing](https://crates.io/crates/tracing), [tracing-subscriber](https://crates.io/crates/tracing-subscriber), [tokio](https://crates.io/crates/tokio) (core features), [once_cell](https://crates.io/crates/once_cell), [parking_lot](https://crates.io/crates/parking_lot) | S2 exit gate |
| **3.1 Independent** | [clap](https://crates.io/crates/clap), [serde_yaml](https://crates.io/crates/serde_yaml)/[serde_yml](https://crates.io/crates/serde_yml), [toml](https://crates.io/crates/toml), [chrono](https://crates.io/crates/chrono)/[time](https://crates.io/crates/time), [humantime](https://crates.io/crates/humantime), [dirs](https://crates.io/crates/dirs), [dotenvy](https://crates.io/crates/dotenvy), [regex](https://crates.io/crates/regex), [fnv](https://crates.io/crates/fnv), [sha2](https://crates.io/crates/sha2), [pem](https://crates.io/crates/pem), [env_logger](https://crates.io/crates/env_logger), [mimalloc](https://crates.io/crates/mimalloc) | 3.0 complete |
| **3.2 Control plane** | [capnp](https://crates.io/crates/capnp), [capnpc](https://crates.io/crates/capnpc), [capnp-rpc](https://crates.io/crates/capnp-rpc), [prost](https://crates.io/crates/prost) | 3.0 complete |
| **3.3 Transport** | [quinn](https://crates.io/crates/quinn)/[quiche](https://crates.io/crates/quiche), [hyper](https://crates.io/crates/hyper), [h2](https://crates.io/crates/h2), [rustls](https://crates.io/crates/rustls)/[boring](https://crates.io/crates/boring), [tokio-rustls](https://crates.io/crates/tokio-rustls)/[tokio-boring](https://crates.io/crates/tokio-boring), [reqwest](https://crates.io/crates/reqwest), [webpki-roots](https://crates.io/crates/webpki-roots), [rustls-native-certs](https://crates.io/crates/rustls-native-certs), [tokio-tungstenite](https://crates.io/crates/tokio-tungstenite), [tower](https://crates.io/crates/tower), [tower-http](https://crates.io/crates/tower-http), [base64](https://crates.io/crates/base64), [http](https://crates.io/crates/http), [http-body](https://crates.io/crates/http-body), [http-body-util](https://crates.io/crates/http-body-util), [crossbeam-channel](https://crates.io/crates/crossbeam-channel) | 3.0 + 3.2 |
| **3.4 Session+datagram** | [dashmap](https://crates.io/crates/dashmap), [arc-swap](https://crates.io/crates/arc-swap), [smallvec](https://crates.io/crates/smallvec), [arrayvec](https://crates.io/crates/arrayvec), [bstr](https://crates.io/crates/bstr), [nom](https://crates.io/crates/nom) (if adopted) | 3.3 complete |
| **3.5 Supervisor** | [ractor](https://crates.io/crates/ractor) (if adopted), [backon](https://crates.io/crates/backon), [signal-hook](https://crates.io/crates/signal-hook)/`tokio::signal`, `CancellationToken` (tokio-util) | 3.3 + 3.4 |
| **3.6 Ingress+proxy** | [socket2](https://crates.io/crates/socket2), [nix](https://crates.io/crates/nix), [ipnet](https://crates.io/crates/ipnet), [surge-ping](https://crates.io/crates/surge-ping) (if adopted), [fast-socks5](https://crates.io/crates/fast-socks5) (if adopted), [hickory-resolver](https://crates.io/crates/hickory-resolver) | 3.3 + 3.5 |
| **3.7 Management** | [axum](https://crates.io/crates/axum), [jsonwebtoken](https://crates.io/crates/jsonwebtoken), [crypto_box](https://crates.io/crates/crypto_box), [x25519-dalek](https://crates.io/crates/x25519-dalek), [ed25519-dalek](https://crates.io/crates/ed25519-dalek), [ssh-key](https://crates.io/crates/ssh-key), [fd-lock](https://crates.io/crates/fd-lock), [open](https://crates.io/crates/open), [tracing-journald](https://crates.io/crates/tracing-journald) | 3.5 + 3.6 |
| **3.8 CLI assembly** | `cloudflare` (if adopted), [flate2](https://crates.io/crates/flate2), [zip](https://crates.io/crates/zip), [sysinfo](https://crates.io/crates/sysinfo), [which](https://crates.io/crates/which), [hostname](https://crates.io/crates/hostname), [notify](https://crates.io/crates/notify) | all prior |
| **3.9 Platform** | [sd-notify](https://crates.io/crates/sd-notify)/[systemd](https://crates.io/crates/systemd), [plist](https://crates.io/crates/plist), [windows-service](https://crates.io/crates/windows-service), [windows-sys](https://crates.io/crates/windows-sys), [winreg](https://crates.io/crates/winreg) | 3.8 complete |
| **Testing (all stages)** | [proptest](https://crates.io/crates/proptest), [mockall](https://crates.io/crates/mockall), [rstest](https://crates.io/crates/rstest), [criterion](https://crates.io/crates/criterion), [tokio-test](https://crates.io/crates/tokio-test), [tempfile](https://crates.io/crates/tempfile), [wiremock](https://crates.io/crates/wiremock), [pretty_assertions](https://crates.io/crates/pretty_assertions), [insta](https://crates.io/crates/insta), [tracing-test](https://crates.io/crates/tracing-test), [portpicker](https://crates.io/crates/portpicker) | per-stage as needed |
| **Tooling (all stages)** | [debtmap-cli](https://crates.io/crates/debtmap-cli), [debtmap](https://crates.io/crates/debtmap) lib | CI from 3.0 onward |

---

## Layer 1 — Runtime and Concurrency

Foundation crates. The runtime choice is the single most pervasive
decision — it colors every other layer.

### 1.1 Async Runtimes

| Crate | Version | T | M | S | Dimension | Span | Flag Path |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [tokio](https://crates.io/crates/tokio) | 1.x | 5 | 5 | 5 | Plumbing | ALL 30 catalogs | `runtime-tokio` |
| [glommio](https://crates.io/crates/glommio) | 0.9.x | 3 | 3 | 3 | Plumbing | ALL 30 catalogs | `runtime-glommio` |

**[tokio](https://crates.io/crates/tokio) — Default path:**

- Multi-threaded work-stealing scheduler
- cloudflared's ~162 goroutines map to [tokio](https://crates.io/crates/tokio) tasks
- Massive ecosystem: [quinn](https://crates.io/crates/quinn), [hyper](https://crates.io/crates/hyper), [tungstenite](https://crates.io/crates/tungstenite), [reqwest](https://crates.io/crates/reqwest) all assume [tokio](https://crates.io/crates/tokio)
- Select loops, channels, timers, signal handlers all have native support
- Risk: ecosystem lock-in

**[glommio](https://crates.io/crates/glommio) — Alternative path:**

- Thread-per-core io_uring runtime (Linux-only)
- Eliminates cross-thread synchronization entirely
- Better I/O throughput for network-heavy workloads
- Caveat: Linux-only (no macOS/Windows), smaller ecosystem
- Would require custom integration layers for QUIC/HTTP libraries
- ADR candidate: evaluate if thread-per-core model fits cloudflared's
  concurrency topology (162 tasks, 4 HA connections)

**Pre-decision: [tokio](https://crates.io/crates/tokio).** [Glommio](https://crates.io/crates/glommio) is Linux-only, incompatible with
the [quinn](https://crates.io/crates/quinn)/[hyper](https://crates.io/crates/hyper)/[reqwest](https://crates.io/crates/reqwest) ecosystem, and would require custom
integration layers for every transport crate. [Tokio](https://crates.io/crates/tokio) is the only
viable choice for cross-platform support. [Glommio](https://crates.io/crates/glommio) prototyping
deferred to S4 if contention benchmarks warrant it.

### 1.2 Async Utilities

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [tokio-util](https://crates.io/crates/tokio-util) | 0.7.x | 5 | 4 | 5 | Plumbing | CON, IT, SS, SM, SES, SUP |
| [tokio-stream](https://crates.io/crates/tokio-stream) | 0.1.x | 5 | 4 | 5 | Plumbing | CON, SS, PRX |
| [futures](https://crates.io/crates/futures) | 0.3.x | 5 | 5 | 5 | Plumbing | CON, IT, PRX |
| [futures-lite](https://crates.io/crates/futures-lite) | 2.x | 4 | 4 | 5 | Plumbing | CON, PRX |
| [async-trait](https://crates.io/crates/async-trait) | 0.1.x | 5 | 5 | 5 | Plumbing | PF (80+ trait sites) |
| [pin-project](https://crates.io/crates/pin-project) | 1.x | 5 | 5 | 5 | Plumbing | PRX, SS, CON |
| [pin-project-lite](https://crates.io/crates/pin-project-lite) | 0.2.x | 5 | 5 | 5 | Plumbing | PRX, SS |

**Notes:**

- [tokio-util](https://crates.io/crates/tokio-util) provides `CancellationToken` (maps 1:1 to Go's
  `context.WithCancel` hierarchy and `graceShutdownC` broadcast)
- [futures](https://crates.io/crates/futures) needed for `try_join!` (errgroup semantics) and stream
  combinators
- [async-trait](https://crates.io/crates/async-trait) still useful on MSRV < 1.75; native async fn in
  traits available on newer compilers
- **Pre-decision: MSRV ≥ 1.75.** New project, no legacy consumers.
  Native async fn in traits eliminates [async-trait](https://crates.io/crates/async-trait) proc-macro
  overhead. [async-trait](https://crates.io/crates/async-trait) retained only for third-party trait compat
- `pin-project-lite` is zero-dep lighter alternative to `pin-project`

### 1.3 Non-Async Concurrency Primitives

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [crossbeam-channel](https://crates.io/crates/crossbeam-channel) | 0.5.x | 5 | 5 | 5 | Plumbing | CON, SS, SUP |
| [crossbeam-utils](https://crates.io/crates/crossbeam-utils) | 0.8.x | 5 | 5 | 5 | Plumbing | CON |
| [crossbeam-epoch](https://crates.io/crates/crossbeam-epoch) | 0.9.x | 5 | 5 | 5 | Plumbing | SS |
| [parking_lot](https://crates.io/crates/parking_lot) | 0.12.x | 5 | 5 | 5 | Plumbing | CON, SS, SES |
| [flume](https://crates.io/crates/flume) | 0.11.x | 4 | 4 | 4 | Plumbing | CON, SS |
| [rayon](https://crates.io/crates/rayon) | 1.x | 5 | 5 | 5 | Plumbing | TST, OBS |

**Notes:**

- [crossbeam-channel](https://crates.io/crates/crossbeam-channel) enables non-async actor-like event loops if we
  decide against full async for some subsystems (e.g., supervisor main
  loop, session manager event loop)
- [parking_lot](https://crates.io/crates/parking_lot) Mutex/RwLock are faster than std for uncontended paths
  and avoid poisoning; maps directly to Go's `sync.Mutex`/`sync.RWMutex`
- [flume](https://crates.io/crates/flume) is an alternative MPSC channel that works both sync and async
- `rayon` for CPU-parallel work (diagnostic artifact collection,
  batch operations) — not on hot path
- Go's v2 session manager uses a single-goroutine event loop pattern
  that could map to [crossbeam-channel](https://crates.io/crates/crossbeam-channel) select rather than async

### 1.4 Synchronization Primitives

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [arc-swap](https://crates.io/crates/arc-swap) | 1.x | 4 | 5 | 5 | Plumbing | SS, CON, CFG |
| [once_cell](https://crates.io/crates/once_cell) | 1.x | 5 | 5 | 5 | Plumbing | PF (7 init sites) |
| [dashmap](https://crates.io/crates/dashmap) | 6.x | 4 | 4 | 4 | Plumbing | SES, SS, CON |
| [hashbrown](https://crates.io/crates/hashbrown) | 0.15.x | 5 | 5 | 5 | Plumbing | SES, SS |

**Notes:**

- [arc-swap](https://crates.io/crates/arc-swap) for atomic pointer swaps on hot-path config (replaces
  benign Go pointer swaps like v2 manager logger)
- [once_cell](https://crates.io/crates/once_cell) replaces Go's 7 `init()` sites and 2 `sync.Once`
  closures; `std::sync::OnceLock` is now stable but [once_cell](https://crates.io/crates/once_cell)
  provides `Lazy` convenience
- [dashmap](https://crates.io/crates/dashmap) for concurrent session registries (replaces
  `sync.RWMutex`-protected maps with sharding)
- `hashbrown` is the Swiss-table HashMap that backs `std::HashMap`
  since Rust 1.36; explicit dep only if we need raw access to
  `RawTable` APIs or `no_std` support

### 1.5 Signal Handling

| Crate | Version | T | M | S | Dimension | Span | Flag Path |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [tokio (signal feature)](https://crates.io/crates/tokio) | 1.x | 5 | 5 | 5 | Plumbing | IT, PLT, CON | `runtime-tokio` |
| [signal-hook](https://crates.io/crates/signal-hook) | 0.3.x | 4 | 5 | 4 | Plumbing | IT, PLT, CON | any runtime |
| [ctrlc](https://crates.io/crates/ctrlc) | 3.x | 4 | 5 | 5 | Plumbing | IT | any runtime |

**Notes:**

- `tokio::signal` is native if we go full [tokio](https://crates.io/crates/tokio) — async
  SIGINT/SIGTERM handling, maps to Go's `signal.Notify()`
- [signal-hook](https://crates.io/crates/signal-hook) is runtime-agnostic — works with [glommio](https://crates.io/crates/glommio), [crossbeam](https://crates.io/crates/crossbeam)
  event loops, or bare threads
- `ctrlc` is simplest, cross-platform Ctrl+C handling
- Go has 3 signal-related goroutines (SIGINT, SIGTERM, stdin control)

### 1.6 Memory Allocators

| Crate | Version | T | M | S | Dimension | Span | Flag Path |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [mimalloc](https://crates.io/crates/mimalloc) | 0.1.x | 4 | 4 | 4 | Plumbing | PRX, TT, SES, CON | `alloc-mimalloc` |

**Notes:**

- [mimalloc](https://crates.io/crates/mimalloc) can reduce allocator contention and fragmentation in
  connection-heavy workloads with many short-lived buffers
- Global allocator choice should remain feature-flagged because this
  changes whole-process behavior (`alloc-system` vs `alloc-mimalloc`)
- Relevant where S1 shows high connection churn, bidirectional copy,
  and datagram frame allocation pressure
- **Pre-decision: feature-flagged optional.** Not ADR-worthy —
  `alloc-mimalloc` feature flag is sufficient. Benchmark in S4

### 1.7 Rate Limiting

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [governor](https://crates.io/crates/governor) | 0.8.x | 4 | 4 | 4 | Domain | FEA, PRX, CON |

**Notes:**

- Token-bucket / GCRA rate limiter — replaces any custom rate
  limiting in connection acceptance, API call throttling, or proxy
  request gating
- Async-aware ([governor](https://crates.io/crates/governor) integrates with [tokio](https://crates.io/crates/tokio) via `clock` feature)
- S1 shows rate-limiting concerns in `flow/limiter` (~200 Go lines)
  and connection-level admission control

### 1.8 Async Caching

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [moka](https://crates.io/crates/moka) | 0.12.x | 4 | 4 | 4 | Domain | FEA, EDG, CFG |

**Notes:**

- Concurrent, async-aware cache with TTL, TTI, and
  size-bounded eviction (TinyLFU-inspired)
- Replaces ad-hoc `sync.Map` or channel-based caching in Go
  (feature flags, edge address pool, API response caching)
- Integrates with [tokio](https://crates.io/crates/tokio) runtime for async `get_with` / `try_get_with`

---

## Layer 2 — Serialization and Data Formats

### 2.1 Core Serialization

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [serde](https://crates.io/crates/serde) | 1.x | 5 | 5 | 5 | Plumbing | CFG, CLI, UAC, OBS, FEA, WP |
| [serde_json](https://crates.io/crates/serde_json) | 1.x | 5 | 5 | 5 | Plumbing | UAC, OBS, FEA, WP, AP |
| [serde_yaml](https://crates.io/crates/serde_yaml) | 0.9.x | 4 | 4 | 4 | Plumbing | CFG, CLI |
| [serde_yml](https://crates.io/crates/serde_yml) | 0.0.x | 3 | 2 | 3 | Plumbing | CFG, CLI |
| [toml](https://crates.io/crates/toml) | 0.8.x | 5 | 5 | 5 | Plumbing | CFG, DEP |
| [capnp](https://crates.io/crates/capnp) | 0.20.x | 4 | 4 | 4 | Behavioral | CAP, EDG, TUN, WP |
| [capnpc](https://crates.io/crates/capnpc) | 0.20.x | 4 | 4 | 4 | Behavioral | CAP (build-time) |
| [capnp-rpc](https://crates.io/crates/capnp-rpc) | 0.20.x | 4 | 3 | 4 | Behavioral | CAP, EDG, TUN |
| [prost](https://crates.io/crates/prost) | 0.13.x | 5 | 5 | 5 | Domain | WP (v2 tracing spans) |

**Notes:**

- [serde_yaml](https://crates.io/crates/serde_yaml) is deprecated upstream; [serde_yml](https://crates.io/crates/serde_yml) is the maintained
  fork but version-immature
- **Pre-decision: [serde_yaml](https://crates.io/crates/serde_yaml) 0.9.x for S3.** Deprecated but
  stable and widely used. Migrate to [serde_yml](https://crates.io/crates/serde_yml) when it reaches
  0.1.x+ maturity. Not ADR-worthy — swap is mechanical
- [toml](https://crates.io/crates/toml) for parity TOML contracts (S3 parity harness), Cargo
  workspace config, and potentially TOML-based config alternative
- [capnp](https://crates.io/crates/capnp) + [capnpc](https://crates.io/crates/capnpc) + [capnp-rpc](https://crates.io/crates/capnp-rpc) are the Cap'n Proto stack —
  maintained by David Renshaw (original Cap'n Proto team); Go schemas
  can be reused directly with [capnpc](https://crates.io/crates/capnpc) codegen; POGS-style adapter
  layer must be hand-written (custom work).
  **lib.rs update:** latest is 0.25.2 (cart lists 0.20.x) — evaluate
  breaking changes before pinning version
- [prost](https://crates.io/crates/prost) for protobuf — datagram v2 uses protobuf for tracing spans
  (`0x03` frame type). **lib.rs note:** passively maintained —
  maintainer expects official `protobuf` crate to supersede; factor
  into long-term dependency planning
- cloudflared config is YAML-native; config hot-reload and CLI flag
  overlay both depend on YAML decode; 5-layer config precedence
  (flags > env > file > defaults > zero)

### 2.2 Data Primitives

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [bytes](https://crates.io/crates/bytes) | 1.x | 5 | 5 | 5 | Plumbing | WP, PRX, TT, SES, PF |
| [uuid](https://crates.io/crates/uuid) | 1.x | 5 | 5 | 5 | Plumbing | SES, TUN, CAP |
| [url](https://crates.io/crates/url) | 2.x | 5 | 5 | 5 | Plumbing | ING, UAC, CFG, EDG |
| [http](https://crates.io/crates/http) | 1.x | 5 | 5 | 5 | Plumbing | PRX, TT, ING, WP, MET |
| [http-body](https://crates.io/crates/http-body) | 1.x | 5 | 5 | 5 | Plumbing | PRX, TT |
| [http-body-util](https://crates.io/crates/http-body-util) | 0.1.x | 5 | 4 | 5 | Plumbing | PRX, TT |
| [mime](https://crates.io/crates/mime) | 0.3.x | 5 | 5 | 5 | Domain | ING, PRX |
| [percent-encoding](https://crates.io/crates/percent-encoding) | 2.x | 5 | 5 | 5 | Domain | UAC, ING |

**Notes:**

- [bytes](https://crates.io/crates/bytes) for zero-copy buffer management — datagram v2/v3 wire
  frames, bidirectional stream relay, HTTP body chunking
- [http](https://crates.io/crates/http) crate provides shared `Request<B>`, `Response<B>`,
  `HeaderMap`, `StatusCode` types used by [hyper](https://crates.io/crates/hyper), [axum](https://crates.io/crates/axum), [reqwest](https://crates.io/crates/reqwest),
  [tower](https://crates.io/crates/tower) — foundational HTTP vocabulary type
- [http-body](https://crates.io/crates/http-body) + [http-body-util](https://crates.io/crates/http-body-util) provide the `Body` trait and
  utilities for streaming HTTP bodies
- `url` replaces Go `net/url` — used in ingress rule matching,
  origin service URLs, API endpoint construction
- [uuid](https://crates.io/crates/uuid) for session IDs and tunnel IDs — Go uses `google/uuid`

### 2.3 Encoding

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [base64](https://crates.io/crates/base64) | 0.22.x | 5 | 5 | 5 | Domain | WP, TT |
| [hex](https://crates.io/crates/hex) | 0.4.x | 5 | 5 | 5 | Domain | CRY, FEA |

**Subjects:** HTTP/2 tunnel header serialization (base64-encoded
metadata in `Cf-Cloudflared-Request-Headers` /
`Cf-Cloudflared-Response-Headers`), hex encoding for hashes

### 2.4 Stack-Allocated Data Structures

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [smallvec](https://crates.io/crates/smallvec) | 1.x | 5 | 5 | 5 | Plumbing | WP, SES, CON |
| [arrayvec](https://crates.io/crates/arrayvec) | 0.7.x | 5 | 5 | 5 | Plumbing | WP, SES |
| [compact_str](https://crates.io/crates/compact_str) | 0.8.x | 4 | 4 | 4 | Plumbing | CFG, ING |
| [tinyvec](https://crates.io/crates/tinyvec) | 1.x | 5 | 5 | 5 | Plumbing | WP |
| [heapless](https://crates.io/crates/heapless) | 0.8.x | 4 | 4 | 4 | Plumbing | WP |
| [bstr](https://crates.io/crates/bstr) | 1.x | 5 | 5 | 5 | Plumbing | PF (150+ byte/string sites) |

**Notes:**

- [smallvec](https://crates.io/crates/smallvec) for header lists, small collections on hot paths
  (datagram frame metadata, connection option lists)
- [arrayvec](https://crates.io/crates/arrayvec) for fixed-capacity collections (QUIC 6-byte signatures,
  PQ curve lists, small enum arrays)
- `compact_str` for SSO (small-string optimization) — ingress rule
  hostnames, header values, config keys
- [bstr](https://crates.io/crates/bstr) for byte-string operations — Go's implicit `[]byte` ↔
  `string` conversion (150+ sites per S1 porting-friction catalog)
  needs careful handling; [bstr](https://crates.io/crates/bstr) bridges the gap
- `heapless` for `no_std`-compatible fixed-size containers if we
  target embedded or constrained environments

### 2.5 Parsing

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [nom](https://crates.io/crates/nom) | 7.x | 5 | 5 | 5 | Domain | WP, SES, TT |
| [winnow](https://crates.io/crates/winnow) | 0.6.x | 4 | 3 | 4 | Domain | WP, SES |

**Notes:**

- [nom](https://crates.io/crates/nom) for combinator-based parsing of custom binary wire formats:
  datagram v2 suffix-muxed frames, datagram v3 prefix-typed frames
  (4 frame types: 0x00–0x03), QUIC stream 6-byte preamble + 2-byte
  version, Cap'n Proto frame boundary detection
- `winnow` is a [nom](https://crates.io/crates/nom) fork with improved error messages and
  streaming support; newer but less ecosystem adoption
- Alternative: manual `bytes::Buf` slicing — simpler for fixed-format
  frames, but [nom](https://crates.io/crates/nom) composes better for nested/variable-length parsers
- ADR candidate: [nom](https://crates.io/crates/nom) vs manual [bytes](https://crates.io/crates/bytes) parsing for wire formats

---

## Layer 3 — Transport

These crates represent the highest-stakes architectural decisions.
The QUIC choice alone touches 8+ catalogs.

### 3.1 QUIC Transport — THE Critical Decision

S1 documents ONE active QUIC library (`quic-go`) but the Rust rewrite
opens TWO viable paths. S1 also shows that datagram v2 and v3 are
both active wire formats with different framing, registration, and
migration semantics.

| Crate | Version | T | M | S | Dimension | Span | Flag Path |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [quinn](https://crates.io/crates/quinn) | 0.11.x | 4 | 4 | 4 | Behavioral | TUN, PRX, TT, SES, EDG, WP, SM, CON | `quic-quinn` |
| [quiche](https://crates.io/crates/quiche) | 0.22.x | 5 | 4 | 5 | Behavioral | (same span) | `quic-quiche` |
| [tokio-quiche](https://github.com/cloudflare/quiche/tree/master/tokio-quiche) | — | 4 | 4 | 4 | Behavioral | (same span) | `quic-quiche` |
| [h3](https://crates.io/crates/h3) | 0.0.x | 4 | 2 | 4 | Behavioral | TT, WP, PRX | either |
| [h3-quinn](https://crates.io/crates/h3-quinn) | 0.0.x | 4 | 2 | 4 | Behavioral | TT, WP | `quic-quinn` |

**[quinn](https://crates.io/crates/quinn) — Pure Rust path:**

- Pure Rust QUIC built on [rustls](https://crates.io/crates/rustls)
- Native [tokio](https://crates.io/crates/tokio) integration, mature async API
- Supports QUIC datagrams (RFC 9221) — critical for v2/v3 wire format
- Active maintainership (>10 contributors, djc + main team)
- Does NOT support BoringSSL backend → FIPS path requires separate
  TLS decision
- No UDP GSO/GRO yet (performance implications)

**quiche — Cloudflare's own:**

- Cloudflare's production QUIC implementation in C
- BoringSSL backend → native FIPS/PQ support
- Used in production at Cloudflare scale (HTTP/3 edge)
- Pure C library with Rust bindings via [quiche](https://crates.io/crates/quiche) crate (maintained
  by Cloudflare themselves)
- `tokio-quiche` is the async [tokio](https://crates.io/crates/tokio) wrapper — part of the same
  `cloudflare/quiche` repository, maintained by the same Cloudflare
  team. Younger than quiche core but actively developed with
  Cloudflare backing
- Supports QUIC datagrams natively
- Provides HTTP/3 support built-in

**h3 — HTTP/3 layer:**

- HTTP/3 implementation on top of any QUIC backend
- `h3-quinn` adapts h3 to [quinn](https://crates.io/crates/quinn)'s QUIC API
- Relevant if edge protocol evolves to HTTP/3 (not current baseline
  but future-proofing)
- **Do not adopt in S3** — pre-alpha (0.0.x), not in 2026.3.0
  baseline. Defer to FC phase

**Critical context from S1:**

- Datagram v2: suffix-muxed type byte, RPC-based session registration
- Datagram v3: prefix-typed binary, migration support, manager/muxer
  model, 4 frame types (0x00–0x03)
- QUIC stream signatures: 6-byte preamble + 2-byte version
  (data: `0A 36 CD 12 A1 3E "01"`, RPC: `52 BB 82 5C DB 65`)
- ALPN: `["argotunnel"]`, SNI: `quic.cftunnel.com`
- UDP socket per connection index (cached/reused for firewall pinhole)
- macOS dual-stack: separate `udp4`/`udp6` for DF bit

**ADR required:** [quinn](https://crates.io/crates/quinn) vs [quiche](https://crates.io/crates/quiche). Factors: FIPS weight, PQ curve
support, datagram API completeness, community health, async
ergonomics, Cloudflare internal alignment.

### 3.2 HTTP/2 Transport

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [hyper](https://crates.io/crates/hyper) | 1.x | 5 | 5 | 5 | Behavioral | PRX, TT, MET, EDG, UAC, ING, WP |
| [hyper-util](https://crates.io/crates/hyper-util) | 0.1.x | 5 | 4 | 5 | Behavioral | PRX, MET, UAC |
| [h2](https://crates.io/crates/h2) | 0.4.x | 5 | 5 | 5 | Behavioral | TT, WP, PRX |

**Notes:**

- [hyper](https://crates.io/crates/hyper) 1.x is the HTTP framework; [h2](https://crates.io/crates/h2) is the low-level HTTP/2
  frame layer (hyper uses h2 internally)
- cloudflared's HTTP/2 transport tunnels requests through HTTP/2
  streams with base64-encoded headers and bidirectional body streaming
- [hyper](https://crates.io/crates/hyper) also serves the metrics HTTP endpoint and management HTTP
  server
- Direct [h2](https://crates.io/crates/h2) access needed for custom frame control in the tunnel
  transport path (stream type detection via
  `Cf-Cloudflared-Proxy-Connection-Upgrade` header)
- SNI: `h2.cftunnel.com` for HTTP/2 edge connections
- HTTP/2 stream types: control-stream, configuration-update,
  websocket, TCP (warp-routing), HTTP (default)

### 3.3 TLS — FIPS and Non-FIPS Paths

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [rustls](https://crates.io/crates/rustls) | 0.23.x | 5 | 5 | 5 | Crypto | CRY, TT, EDG, WP, PS |
| [rustls-pemfile](https://crates.io/crates/rustls-pemfile) | 2.x | 5 | 5 | 5 | Crypto | CRY |
| [tokio-rustls](https://crates.io/crates/tokio-rustls) | 0.26.x | 5 | 5 | 5 | Crypto | TT, EDG |
| [boring](https://crates.io/crates/boring) | 4.x | 4 | 4 | 4 | Crypto | CRY, TT, WP, PS |
| [boring-sys](https://crates.io/crates/boring-sys) | 4.x | 4 | 3 | 4 | Crypto | CRY |
| [tokio-boring](https://crates.io/crates/tokio-boring) | 4.x | 4 | 3 | 4 | Crypto | TT, EDG |
| [webpki-roots](https://crates.io/crates/webpki-roots) | 0.26.x | 5 | 5 | 5 | Crypto | CRY |
| [rustls-native-certs](https://crates.io/crates/rustls-native-certs) | 0.8.x | 5 | 4 | 5 | Crypto | CRY, PLT |
| [rustls-platform-verifier](https://crates.io/crates/rustls-platform-verifier) | 0.5.x | 4 | 4 | 4 | Crypto | CRY, PLT |

**Notes:**

- Go cloudflared uses `crypto/tls` with `fipsonly` build tag and PQ
  curves (X25519MLKEM768 non-FIPS, P256Kyber768Draft00 FIPS)
- **Non-FIPS:** [rustls](https://crates.io/crates/rustls) — audited (Cure53 + Alpha-Omega), pure
  Rust, no C dependencies
- **FIPS:** [boring](https://crates.io/crates/boring) — Rust bindings to BoringSSL, FIPS-validated
  crypto module with PQ curves
- Both paths via Cargo features (`--features fips`)
- [quinn](https://crates.io/crates/quinn) uses [rustls](https://crates.io/crates/rustls) natively; FIPS+QUIC needs either [quinn](https://crates.io/crates/quinn) with
  [boring](https://crates.io/crates/boring) backend (experimental) or [quiche](https://crates.io/crates/quiche) (native BoringSSL)
- [webpki-roots](https://crates.io/crates/webpki-roots) provides Mozilla CA bundle;
  [rustls-native-certs](https://crates.io/crates/rustls-native-certs) reads system CA store
- Go's CA assembly: system pool → Cloudflare embedded CAs (3 certs) →
  hello cert → `--origin-ca-pool`
- Windows CA quirk: `x509.SystemCertPool()` fails in Go (issue #16736);
  [rustls-native-certs](https://crates.io/crates/rustls-native-certs) handles this correctly
- `rustls-platform-verifier` delegates certificate verification to the
  OS platform verifier (Security.framework on macOS, CryptoAPI on
  Windows, OpenSSL on Linux) — more correct than [webpki-roots](https://crates.io/crates/webpki-roots) for
  enterprise environments with custom CA roots

### 3.4 HTTP Client

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [reqwest](https://crates.io/crates/reqwest) | 0.12.x | 5 | 5 | 4 | Behavioral | UAC, AP, EDG, DEP, FEA |
| [ureq](https://crates.io/crates/ureq) | 3.x | 4 | 4 | 4 | Domain | UAC, DEP |
| [cloudflare](https://crates.io/crates/cloudflare) | 0.14.x | 4 | 4 | 4 | Domain | UAC, AP, DEP |

**Notes:**

- [reqwest](https://crates.io/crates/reqwest) replaces Go's `net/http` client for Cloudflare API calls:
  tunnel CRUD, route management, virtual network ops, hostname routing,
  quick-tunnel provisioning, token flows, auto-update check
- Supports [rustls](https://crates.io/crates/rustls) and [boring](https://crates.io/crates/boring) TLS backends via features
- Built on [hyper](https://crates.io/crates/hyper) — consistent TLS stack
- [ureq](https://crates.io/crates/ureq) is a blocking HTTP client — useful if some codepaths avoid
  async (e.g., startup credential fetch before runtime init)
- `cloudflare` crate is an SDK candidate for Cloudflare API surfaces;
  evaluate endpoint coverage against cloudflared-specific operations
  before adoption (many projects still need bespoke endpoints)
- cfapi envelope pattern: `{success, errors, result, result_info}`
  with auto-pagination (stop when `Count < PerPage`)

### 3.5 WebSocket

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [tokio-tungstenite](https://crates.io/crates/tokio-tungstenite) | 0.26.x | 4 | 4 | 4 | Behavioral | EDG, UAC, PRX, FEA, WP |
| [tungstenite](https://crates.io/crates/tungstenite) | 0.26.x | 4 | 5 | 4 | Behavioral | PRX, WP |
| [fastwebsockets](https://crates.io/crates/fastwebsockets) | 0.8.x | 4 | 3 | 4 | Behavioral | PRX |

**Notes:**

- Replaces `gorilla/websocket` — management WebSocket log streaming,
  carrier WebSocket proxying, real-time event streams
- `tungstenite` is the sync core; [tokio-tungstenite](https://crates.io/crates/tokio-tungstenite) wraps it async
- `fastwebsockets` is a high-performance alternative from Deno team —
  lower overhead for proxy hot path
- Management WS: idle 5min, heartbeat 15s, 1 concurrent session,
  same-actor preemption
- WebSocket accept-key: SHA-1 per RFC 6455

### 3.6 Service/Middleware Framework

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [tower](https://crates.io/crates/tower) | 0.5.x | 5 | 5 | 5 | Facade | PRX, ING, MET, OBS, SUP |
| [tower-http](https://crates.io/crates/tower-http) | 0.6.x | 5 | 4 | 5 | Facade | PRX, ING, MET |
| [tower-layer](https://crates.io/crates/tower-layer) | 0.3.x | 5 | 5 | 5 | Facade | ALL composable |

**Notes:**

- `tower::Service` trait is the composable middleware/service
  abstraction — enables layering metrics, timeouts, retries,
  rate-limiting, [tracing](https://crates.io/crates/tracing), concurrency limits as generic middleware
- If we build our own concurrency cadence (deviating from Go impl),
  [tower](https://crates.io/crates/tower) gives us a well-tested service composition framework
- [tower-http](https://crates.io/crates/tower-http) provides ready-made HTTP middleware (CORS, compression,
  trace, timeout, auth)
- cloudflared's management server has CORS (`*.cloudflare.com`, 300s
  max-age) — [tower-http](https://crates.io/crates/tower-http) CorsLayer handles this directly
- ADR candidate: tower-based architecture vs flat function dispatch

### 3.7 Cloudflare Transport Ecosystem

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [pingora](https://crates.io/crates/pingora) | 0.4.x | 4 | 3 | 4 | Behavioral | PRX, TT, ING, OBS |

**Notes:**

- `pingora` is a high-performance proxy stack from Cloudflare and is a
  strategic candidate even if we do not adopt it in S3
- Most realistic usage is selective borrowing (pooling, protocol glue,
  middleware ideas) rather than full replacement of cloudflared paths
- Keep as a shopping-cart option to avoid prematurely excluding
  Cloudflare-aligned transport components

---

## Layer 4 — Observability

### 4.1 Logging

| Crate | Version | T | M | S | Dimension | Span | Flag Path |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [tracing](https://crates.io/crates/tracing) | 0.1.x | 5 | 5 | 5 | Plumbing | OBS, EP, IT, ALL | `logging-native` |
| [tracing-subscriber](https://crates.io/crates/tracing-subscriber) | 0.3.x | 5 | 5 | 5 | Plumbing | OBS, IT | `logging-native` |
| [tracing-appender](https://crates.io/crates/tracing-appender) | 0.2.x | 5 | 4 | 5 | Plumbing | OBS | `logging-native` |
| [log](https://crates.io/crates/log) | 0.4.x | 5 | 5 | 5 | Facade | OBS | `logging-compat` |
| [tracing-log](https://crates.io/crates/tracing-log) | 0.2.x | 5 | 4 | 5 | Facade | OBS | bridge |
| [env_logger](https://crates.io/crates/env_logger) | 0.11.x | 5 | 5 | 5 | Plumbing | OBS | `logging-compat` |
| [tracing-journald](https://crates.io/crates/tracing-journald) | 0.3.x | 4 | 4 | 4 | Platform | OBS, PLT, DEP | linux/systemd |
| [tracing-error](https://crates.io/crates/tracing-error) | 0.2.x | 4 | 4 | 4 | Plumbing | OBS, EP | `logging-native` |

**Notes:**

- Go uses `zerolog` — structured JSON logging with level filtering
- **`logging-native` path:** [tracing](https://crates.io/crates/tracing) provides structured,
  span-aware logging with async awareness — richer than zerolog
- **`logging-compat` path:** We may build our own logging facade that
  outputs zerolog-compatible JSON for upstream log streaming while
  using [tracing](https://crates.io/crates/tracing) internally for local diagnostics
- [tracing-log](https://crates.io/crates/tracing-log) bridges the `log` crate to [tracing](https://crates.io/crates/tracing) — allows
  third-party crates using `log` to emit into our [tracing](https://crates.io/crates/tracing) pipeline
- [tracing-journald](https://crates.io/crates/tracing-journald) is the direct path for Linux/systemd deployments
  that want structured logs in journald without text re-parsing
- Management WebSocket log streaming requires a custom
  [tracing-subscriber](https://crates.io/crates/tracing-subscriber) `Layer` that serializes events to the WS
  client — this is custom build regardless of path
- `ConnAwareLogger` from S1: chooses warn/error based on active
  connection count — custom `Layer` implementation
- `tracing-error` enriches error types with `SpanTrace` context —
  captures the active [tracing](https://crates.io/crates/tracing) span stack when an error is created,
  bridging the observability and error-propagation catalogs

### 4.2 Distributed Tracing

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [opentelemetry](https://crates.io/crates/opentelemetry) | 0.29.x | 4 | 3 | 4 | Plumbing | OBS |
| [opentelemetry-otlp](https://crates.io/crates/opentelemetry-otlp) | 0.29.x | 4 | 3 | 4 | Plumbing | OBS |
| [opentelemetry_sdk](https://crates.io/crates/opentelemetry_sdk) | 0.29.x | 4 | 3 | 4 | Plumbing | OBS |
| [tracing-opentelemetry](https://crates.io/crates/tracing-opentelemetry) | 0.29.x | 4 | 3 | 4 | Plumbing | OBS |

**Notes:**

- cloudflared exports OTEL traces and propagates trace identity
  through tunnel headers
- Pre-1.0 but the Rust OTEL SDK is actively developed by the official
  opentelemetry-rust project
- [tracing-opentelemetry](https://crates.io/crates/tracing-opentelemetry) bridges [tracing](https://crates.io/crates/tracing) spans to OTEL spans
- datagram v2 uses protobuf tracing spans (frame type `0x03`)

### 4.3 Metrics

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [prometheus-client](https://crates.io/crates/prometheus-client) | 0.23.x | 4 | 4 | 4 | Plumbing | MET, TUN, SUP, PRX, SES, TT |
| [metrics](https://crates.io/crates/metrics) | 0.24.x | 4 | 4 | 4 | Facade | MET |
| [metrics-exporter-prometheus](https://crates.io/crates/metrics-exporter-prometheus) | 0.16.x | 4 | 4 | 4 | Domain | MET |
| [prometheus](https://crates.io/crates/prometheus) | 0.13.x | 4 | 4 | 4 | Plumbing | MET |

**Notes:**

- [prometheus-client](https://crates.io/crates/prometheus-client) is the official Prometheus Rust client — closest
  match to Go's `prometheus/client_golang`
- [metrics](https://crates.io/crates/metrics) is a facade crate (like `log` for logging) with pluggable
  exporters — `metrics-exporter-prometheus` provides Prometheus output
- [prometheus](https://crates.io/crates/prometheus) (tikv-maintained) is older, heavier API
- cloudflared registers counters/gauges across connection, quic,
  supervisor, datagramsession, flow, ingress, tunnelrpc
- Metrics HTTP server needs an HTTP framework (see 3.2, 7.6)
- ADR candidate: [prometheus-client](https://crates.io/crates/prometheus-client) vs [metrics](https://crates.io/crates/metrics) facade

### 4.4 Error Reporting

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [sentry](https://crates.io/crates/sentry) | 0.35.x | 4 | 4 | 4 | Domain | EP, OBS |

**Notes:**

- Go cloudflared reports panics to Sentry (bidirectional stream pipe
  pattern 5 — panic recovery with stale pipe detection)
- `sentry` crate has native Rust integration with backtrace capture

---

## Layer 5 — Error Handling

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [thiserror](https://crates.io/crates/thiserror) | 2.x | 5 | 5 | 5 | Plumbing | EP, PF, ALL library crates |
| [anyhow](https://crates.io/crates/anyhow) | 1.x | 5 | 5 | 5 | Plumbing | EP, CLI (binary crate only) |
| [miette](https://crates.io/crates/miette) | 7.x | 4 | 4 | 4 | Domain | CLI, OBS |
| [eyre](https://crates.io/crates/eyre) | 0.6.x | 4 | 4 | 4 | Plumbing | EP |

**Notes:**

- [thiserror](https://crates.io/crates/thiserror) for all library crate error enums (6 error families
  from S1: connection, supervisor, RPC, edge-discovery, datagram,
  session)
- [anyhow](https://crates.io/crates/anyhow) restricted to binary crate boundaries only
- [miette](https://crates.io/crates/miette) provides rich diagnostic output (source spans, suggestions)
  — useful for CLI error display to users
- [eyre](https://crates.io/crates/eyre) is an [anyhow](https://crates.io/crates/anyhow) alternative with custom report handlers —
  integrates with [color-eyre](https://crates.io/crates/color-eyre) for backtraces
- Go's `(error, recoverable bool)` dual-return pattern maps to
  `Result<T, E>` with recoverable/fatal variants in error enums
- 5 type-switch sites and 15+ error types require enum taxonomy
- Error aggregation: multiple errors concatenated with `"; "`
  separator in Go API

---

## Layer 6 — CLI and Configuration

### 6.1 CLI Framework

| Crate | Version | T | M | S | Dimension | Span | Flag Path |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [clap](https://crates.io/crates/clap) | 4.x | 5 | 5 | 5 | Plumbing | CLI, CFG, DEP, FEA | `cli-native` |

**Notes:**

- Replaces Go's `urfave/cli/v2` — command families: tunnel, access,
  tail, management, update, proxydns, service install/uninstall
- [clap](https://crates.io/crates/clap) derive macros for subcommand trees; handles env-var fallbacks
  and config-file integration
- **`cli-compat` path:** We may build a thin CLI framework that matches
  `urfave/cli` cadence exactly (flag precedence, help format, error
  messages) to ensure behavioral parity during S4 testing — this
  would be custom work wrapping [clap](https://crates.io/crates/clap) or using a simpler foundation
- ADR candidate: full [clap](https://crates.io/crates/clap) derive vs compat wrapper

### 6.2 Time and Duration

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [chrono](https://crates.io/crates/chrono) | 0.4.x | 5 | 5 | 4 | Plumbing | CFG, OBS, SES, AP |
| [time](https://crates.io/crates/time) | 0.3.x | 5 | 5 | 5 | Plumbing | CFG, OBS |
| [humantime](https://crates.io/crates/humantime) | 2.x | 4 | 5 | 5 | Domain | CLI, CFG |

**Notes:**

- [chrono](https://crates.io/crates/chrono) for timestamp formatting, timezone handling, duration
  parsing — Go uses `time.Duration`, `time.Now()`, `time.After()`,
  `time.NewTicker()` extensively
- [time](https://crates.io/crates/time) is an alternative to [chrono](https://crates.io/crates/chrono) — smaller, `#![forbid(unsafe)]`
- **Pre-decision: [chrono](https://crates.io/crates/chrono) 0.4.x.** Broader API coverage for
  cloudflared's timestamp/timezone needs. [time](https://crates.io/crates/time) kept as optional
  for crates that prefer it. Not ADR-worthy — isolate behind
  time utility module
- [humantime](https://crates.io/crates/humantime) for human-readable duration parsing (e.g., `"5s"`,
  `"210s"`, `"1h"`) — useful for config values
- Key timeouts from S1: idle 210s, RPC 5s, dial edge 15s, heartbeat
  15s, management idle 5min, feature refresh 1h, registration
  interval 1s

### 6.3 Environment and Paths

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [dirs](https://crates.io/crates/dirs) | 6.x | 5 | 5 | 5 | Platform | CFG, DEP, HI |
| [dotenvy](https://crates.io/crates/dotenvy) | 0.15.x | 4 | 4 | 4 | Domain | CFG, CE |

**Notes:**

- [dirs](https://crates.io/crates/dirs) for platform-specific directory discovery: XDG on Linux,
  `~/Library` on macOS, `%AppData%` on Windows
- Config search order: `~/.cloudflared` → `~/.cloudflare-warp` →
  `~/cloudflare-warp` → `/etc/cloudflared` (unix) →
  `/usr/local/etc/cloudflared` (unix) → `$CFDPATH` (Windows) →
  `%ProgramFiles(x86)%` (Windows)
- [dotenvy](https://crates.io/crates/dotenvy) for `.env` file loading (development convenience)

---

## Layer 7 — Domain-Specific

### 7.1 DNS Resolution

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [hickory-resolver](https://crates.io/crates/hickory-resolver) | 0.25.x | 4 | 4 | 4 | Domain | EDG, CFG, ING, FEA |
| [hickory-client](https://crates.io/crates/hickory-client) | 0.25.x | 4 | 4 | 4 | Domain | ING (proxydns) |

**Notes:**

- Formerly `trust-dns` — renamed to `hickory-dns` ecosystem
- Edge discovery: SRV + A record lookup for edge address pool
- Feature flags: TXT record lookup with FNV-32a hash percentage
  rollout
- DNS origin resolution in ingress rules
- `hickory-client` needed for the `proxydns` subcommand which acts
  as a local DNS proxy forwarding to Cloudflare recursive resolver
  (`2606:4700:0cf1:2000::1:53`)

### 7.2 File Watching

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [notify](https://crates.io/crates/notify) | 7.x | 4 | 5 | 4 | Domain | CFG, HI |

**Notes:**

- Replaces Go's `fsnotify` — config YAML hot-reload, credential file
  change detection
- Cross-platform: inotify (Linux), kqueue (macOS),
  ReadDirectoryChanges (Windows)

### 7.3 IP/Network

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [ipnet](https://crates.io/crates/ipnet) | 2.x | 5 | 5 | 5 | Domain | AP, ING |

**Subjects:** IP allow/deny rule evaluation, CIDR matching for ingress
rules and access policies

### 7.4 Retry/Backoff

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [backon](https://crates.io/crates/backon) | 1.x | 4 | 4 | 4 | Domain | SUP, SM, SS |
| [backoff](https://crates.io/crates/backoff) | 0.4.x | 3 | 3 | 3 | Domain | SUP |

**Notes:**

- [backon](https://crates.io/crates/backon) is the modern retry library, 1.0 stable, works async and
  sync
- Go's `retry/backoffhandler` has custom jitter and max-retry logic;
  supervisor FSM: exponential backoff, resets if `resetDeadline` passed
- ADR candidate: [backon](https://crates.io/crates/backon) vs custom backoff (Go's is ~200 lines)

### 7.5 Regex

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [regex](https://crates.io/crates/regex) | 1.x | 5 | 5 | 5 | Domain | ING |

**Subjects:** Ingress rule path matching, origin service URL pattern
matching, management log filter parsing

### 7.6 HTTP Server (Management)

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [axum](https://crates.io/crates/axum) | 0.8.x | 5 | 4 | 5 | Domain | UAC, MET, OBS |

**Notes:**

- For management HTTP server: `/ping`, `/host_details` (JSON),
  `/metrics` (Prometheus), `/debug/pprof/*`, `/logs` (WebSocket)
- Built on [hyper](https://crates.io/crates/hyper) + [tokio](https://crates.io/crates/tokio) + [tower](https://crates.io/crates/tower) — routing, middleware, handler
  ergonomics
- CORS: `*.cloudflare.com`, 300s max-age
- Session gating: 1 concurrent management session, preemption allowed
- Alternative: raw [hyper](https://crates.io/crates/hyper) — management server is relatively simple
- ADR candidate: [axum](https://crates.io/crates/axum) vs raw [hyper](https://crates.io/crates/hyper)

### 7.7 SOCKS5

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [fast-socks5](https://crates.io/crates/fast-socks5) | 0.9.x | 3 | 3 | 3 | Domain | PRX |

**Alternative:** Custom implementation — Go's `socks` package is ~400
lines, well-understood protocol.

### 7.8 ICMP

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [surge-ping](https://crates.io/crates/surge-ping) | 0.8.x | 3 | 3 | 3 | Domain | ING, PRX, PLT, PS |

**Alternative:** Custom raw-socket ICMP using [socket2](https://crates.io/crates/socket2).

**Notes:**

- cloudflared has OS-specific ICMP: Linux (raw socket with
  `/proc/net/ipv4/ping_group_range`), macOS (echo ID tracking),
  Windows (ICMP API via `icmp.SendEcho`)
- 2 ICMP router goroutines (ipv4 + ipv6) at process scope
- Platform-divergent enough that custom impl may be better

### 7.9 Actor Framework

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [ractor](https://crates.io/crates/ractor) | 0.13.x | 3 | 3 | 3 | Behavioral | OVW, SUP, SES |
| [actix](https://crates.io/crates/actix) | 0.13.x | 4 | 4 | 4 | Behavioral | OVW, SUP |
| [kameo](https://crates.io/crates/kameo) | 0.13.x | 3 | 2 | 3 | Behavioral | OVW, SUP |

**Notes:**

- Go's overwatch (`AppManager`) is a simple service registry with
  config-driven replacement — functions as a lightweight actor system
- [ractor](https://crates.io/crates/ractor) provides typed actors with supervision trees — good match
  for supervisor/tunnel/connection hierarchy
- `actix` is more mature but heavier (ships its own runtime)
- `kameo` is newer, built on [tokio](https://crates.io/crates/tokio), smaller API surface
- ADR candidate: actor framework vs custom service registry; Go's
  overwatch is ~300 lines but the supervision semantics (restart
  policy, graceful shutdown, error escalation) benefit from a
  well-tested library

### 7.10 Compression

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [flate2](https://crates.io/crates/flate2) | 1.x | 5 | 5 | 5 | Domain | WP, DEP |
| [zip](https://crates.io/crates/zip) | 2.x | 4 | 4 | 4 | Domain | OBS |

**Notes:**

- [flate2](https://crates.io/crates/flate2) for gzip compression (HTTP response compression, artifact
  compression)
- [zip](https://crates.io/crates/zip) for diagnostic artifact bundle assembly
- `CompressionQuality` field in QUIC `RegisterConnection` request

### 7.11 Cloudflare Foundation Libraries

| Crate | Source | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [foundations](https://github.com/cloudflare/foundations) | GitHub | 4 | 3 | 4 | Facade | OBS, CON, EP, DEP |

**Notes:**

- `cloudflare/foundations` is a serious candidate for production
  service scaffolding (telemetry, config, runtime conventions,
  reliability guardrails)
- This is not a direct crates.io baseline decision; treat it as a
  source-level dependency family to evaluate in ADRs
- The repo should stay in the shopping cart because it can reduce
  custom glue in observability and runtime bootstrap

### 7.12 Auto-Update

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [self_update](https://crates.io/crates/self_update) | 0.41.x | 3 | 3 | 3 | Domain | DEP, PLT, PS |

**Notes:**

- Replaces Go's custom auto-update mechanism (download new binary,
  replace self, restart)
- Go cloudflared: checks Cloudflare API for latest version, downloads
  `.tgz`/`.msi`, replaces binary, exits with code 11 for systemd
  restart
- `self_update` provides GitHub-release and custom-URL update
  backends
- Platform-specific: Linux (replace binary + systemd restart), macOS
  (brew upgrade path + launchd), Windows (MSI with `wusa.exe`)
- Update suppression rules: skip if manual install, timer-based daily
  check, lock-file coordination
- Build-vs-Buy consideration favors custom due to platform divergence
  and update suppression logic (~400 lines)

### 7.13 Runtime Debugging

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [tokio-console](https://crates.io/crates/tokio-console) | 0.1.x | 4 | 3 | 4 | Testing | CON, OBS, TST |
| [console-subscriber](https://crates.io/crates/console-subscriber) | 0.4.x | 4 | 3 | 4 | Testing | CON, OBS, TST |

**Notes:**

- `tokio-console` is a diagnostic tool for inspecting async runtime
  behavior (task scheduling, waker events, resource contention)
- `console-subscriber` is the [tracing](https://crates.io/crates/tracing) Layer that feeds data to
  tokio-console
- Critical for S4 performance debugging: identifying task starvation,
  excessive wakeups, or lock contention in the async runtime
- Should be feature-gated (`debug-console`) to avoid production
  overhead

---

## Layer 8 — Crypto and Security

Beyond TLS (Layer 3.3), these crates handle specific crypto operations.

### 8.1 Core Crypto Primitives

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [ring](https://crates.io/crates/ring) | 0.17.x | 5 | 5 | 5 | Crypto | CRY, AP |
| [sha2](https://crates.io/crates/sha2) | 0.10.x | 5 | 5 | 5 | Crypto | CRY, CFG |
| [sha1](https://crates.io/crates/sha1) | 0.10.x | 5 | 5 | 5 | Crypto | WP (WebSocket) |
| [hmac](https://crates.io/crates/hmac) | 0.12.x | 5 | 5 | 5 | Crypto | CRY |
| [digest](https://crates.io/crates/digest) | 0.10.x | 5 | 5 | 5 | Crypto | CRY |

**Notes:**

- [ring](https://crates.io/crates/ring) is transitive through [rustls](https://crates.io/crates/rustls); general-purpose crypto:
  HMAC, SHA-256, ECDSA, random bytes
- [sha2](https://crates.io/crates/sha2), `sha1`, [hmac](https://crates.io/crates/hmac), `digest` are from the
  [RustCrypto](https://github.com/RustCrypto) project — pure Rust,
  well-audited, pluggable trait-based design
- `sha1` specifically needed for RFC 6455 WebSocket accept-key
  computation
- SHA-256 used for config fingerprinting and build hash
- ADR consideration: [ring](https://crates.io/crates/ring) primitives vs RustCrypto crates — ring
  is C-backed (BoringSSL-derived), RustCrypto is pure Rust

### 8.2 Elliptic Curve and Signatures

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [p256](https://crates.io/crates/p256) | 0.13.x | 5 | 5 | 5 | Crypto | CRY |
| [ecdsa](https://crates.io/crates/ecdsa) | 0.16.x | 5 | 5 | 5 | Crypto | CRY |
| [elliptic-curve](https://crates.io/crates/elliptic-curve) | 0.13.x | 5 | 5 | 5 | Crypto | CRY |
| [ed25519-dalek](https://crates.io/crates/ed25519-dalek) | 2.x | 5 | 5 | 5 | Crypto | CRY |
| [rand](https://crates.io/crates/rand) | 0.8.x | 5 | 5 | 5 | Crypto | CRY, SES |

**Notes:**

- [p256](https://crates.io/crates/p256) + [ecdsa](https://crates.io/crates/ecdsa) for ECDSA P-256 key generation (SSH cert gen)
- [ed25519-dalek](https://crates.io/crates/ed25519-dalek) as alternative signing algorithm
- `rand` for secure random generation (session IDs, nonces) — Go
  uses `crypto/rand`
- All from RustCrypto project

### 8.3 Certificate and Key Handling

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [pem](https://crates.io/crates/pem) | 3.x | 4 | 5 | 5 | Crypto | CRY |
| [x509-parser](https://crates.io/crates/x509-parser) | 0.16.x | 4 | 4 | 4 | Crypto | CRY |
| [x509-cert](https://crates.io/crates/x509-cert) | 0.2.x | 5 | 4 | 5 | Crypto | CRY |
| [der](https://crates.io/crates/der) | 0.7.x | 5 | 5 | 5 | Crypto | CRY |
| [pkcs8](https://crates.io/crates/pkcs8) | 0.10.x | 5 | 5 | 5 | Crypto | CRY |

**Notes:**

- [pem](https://crates.io/crates/pem) for PEM file parsing (origin cert, credential files) —
  Go uses `encoding/pem`
- `x509-parser` for certificate chain inspection, field extraction
- `x509-cert` from RustCrypto — newer, trait-based, composable
- `der` + `pkcs8` for low-level ASN.1/DER encoding and private key
  format handling
- Certificate reloader: `tlsconfig/certreloader` — reload on file
  change (pairs with [notify](https://crates.io/crates/notify) crate from 7.2)

### 8.4 JWT/JOSE

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [jsonwebtoken](https://crates.io/crates/jsonwebtoken) | 9.x | 4 | 5 | 4 | Crypto | AP, FEA |
| [josekit](https://crates.io/crates/josekit) | 0.12.x | 3 | 3 | 3 | Crypto | AP |

**Notes:**

- cloudflared validates JWT tokens for Access policies (AUD claim,
  OIDC verifier against Cloudflare endpoints)
- [jsonwebtoken](https://crates.io/crates/jsonwebtoken) is simpler, well-maintained, sufficient for JWT
  validation (RS256)
- `josekit` is full JOSE (JWE, JWK, JWS) — heavier but more complete
- Go uses `go-jose/go-jose/v4` for JWT handling

### 8.5 Sealed Box / NaCl

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [crypto_box](https://crates.io/crates/crypto_box) | 0.9.x | 4 | 4 | 4 | Crypto | AP, CRY |
| [xsalsa20poly1305](https://crates.io/crates/xsalsa20poly1305) | 0.9.x | 4 | 4 | 4 | Crypto | AP |
| [curve25519-dalek](https://crates.io/crates/curve25519-dalek) | 4.x | 5 | 5 | 5 | Crypto | AP, CRY |
| [x25519-dalek](https://crates.io/crates/x25519-dalek) | 2.x | 5 | 5 | 5 | Crypto | AP |

**Notes:**

- Replaces Go's `nacl/box` — Curve25519 + XSalsa20-Poly1305 sealed
  box for access token transfer
- [crypto_box](https://crates.io/crates/crypto_box) is the high-level API from RustCrypto
- `xsalsa20poly1305` is the cipher primitive
- `curve25519-dalek` / [x25519-dalek](https://crates.io/crates/x25519-dalek) for Diffie-Hellman key exchange
- All pure Rust, part of RustCrypto project

### 8.6 SSH Key Generation

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [ssh-key](https://crates.io/crates/ssh-key) | 0.6.x | 4 | 4 | 4 | Crypto | CRY |

**Subjects:** SSH key generation for short-lived certificate access;
ECDSA P-256 via [ssh-key](https://crates.io/crates/ssh-key) + [p256](https://crates.io/crates/p256) crates

### 8.7 Hashing

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [fnv](https://crates.io/crates/fnv) | 1.x | 5 | 5 | 5 | Domain | FEA |

**Subjects:** FNV-32a hashing for feature flag percentage rollout;
`switchThreshold(accountTag)` produces deterministic percentile
[0, 100) for canary decisions

---

## Layer 9 — Platform

OS/build-target conditional compilation. Heavy use of `#[cfg()]`.

### 9.1 Unix/POSIX

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [nix](https://crates.io/crates/nix) | 0.29.x | 5 | 5 | 4 | Platform | HI, PLT, PS, CON, ING |
| [libc](https://crates.io/crates/libc) | 0.2.x | 5 | 5 | 5 | Platform | HI, PLT, PS |

**Notes:**

- [nix](https://crates.io/crates/nix) is the high-level safe wrapper over libc — covers socket
  options, process management, signal handling, file descriptors,
  ping-group detection, DF bit setting
- [nix](https://crates.io/crates/nix) overlaps many domains: ICMP raw sockets, QUIC UDP socket
  tuning, service management (fork/exec), systemd notification
- Explicit ICMP overlap: `nix::sys::socket` and related helpers can
  own much of Linux ICMP plumbing instead of splitting logic between
  bespoke syscall wrappers and higher-level ping crates
- [libc](https://crates.io/crates/libc) is the raw FFI bindings — needed where [nix](https://crates.io/crates/nix) doesn't wrap

### 9.2 Systemd

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [sd-notify](https://crates.io/crates/sd-notify) | 0.4.x | 3 | 4 | 4 | Platform | PLT, IT, PS |
| [systemd](https://crates.io/crates/systemd) | 0.10.x | 3 | 3 | 3 | Platform | PLT, IT, PS, DEP |

**Notes:**

- [sd-notify](https://crates.io/crates/sd-notify) for readiness notification (`READY=1`, `STOPPING=1`)
- [systemd](https://crates.io/crates/systemd) crate provides broader integration: journal logging,
  socket activation, service management APIs
- cloudflared's systemd path: service unit + daily update timer;
  auto-update exit code 11 triggers restart policy
- ADR candidate: [sd-notify](https://crates.io/crates/sd-notify) (minimal) vs [systemd](https://crates.io/crates/systemd) (comprehensive)

### 9.3 macOS

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [plist](https://crates.io/crates/plist) | 1.x | 4 | 5 | 4 | Platform | PLT, DEP, PS |

**Notes:**

- launchd plist generation: `com.cloudflare.cloudflared.plist`
- Root installs to `/Library/LaunchDaemons/`, user to
  `~/Library/LaunchAgents/`

### 9.4 Windows

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [windows-service](https://crates.io/crates/windows-service) | 0.7.x | 4 | 4 | 4 | Platform | PLT, HI, DEP, PS |
| [windows-sys](https://crates.io/crates/windows-sys) | 0.59.x | 5 | 5 | 5 | Platform | PLT, PS |
| [winreg](https://crates.io/crates/winreg) | 0.55.x | 4 | 4 | 4 | Platform | PLT, DEP |

**Notes:**

- [windows-service](https://crates.io/crates/windows-service) for SCM registration, install/uninstall, service
  event loop; recovery policy: `SC_ACTION_RESTART` after 20s, 24h
  reset period
- [windows-sys](https://crates.io/crates/windows-sys) for raw Win32 API (WMI queries, `tracert` format,
  ICMP API `icmp.SendEcho`)
- [winreg](https://crates.io/crates/winreg) for Windows Registry operations (service paths, product
  GUIDs)

### 9.5 Cross-Platform

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [socket2](https://crates.io/crates/socket2) | 0.5.x | 5 | 5 | 5 | Platform | PLT, PS, ING, PRX |
| [sysinfo](https://crates.io/crates/sysinfo) | 0.33.x | 4 | 4 | 4 | Platform | OBS, HI, PS |
| [open](https://crates.io/crates/open) | 5.x | 4 | 5 | 4 | Platform | AP, PLT, PS |
| [fd-lock](https://crates.io/crates/fd-lock) | 4.x | 4 | 4 | 4 | Platform | AP, HI |
| [which](https://crates.io/crates/which) | 7.x | 4 | 5 | 4 | Platform | OBS, PLT |
| [hostname](https://crates.io/crates/hostname) | 0.4.x | 4 | 4 | 4 | Platform | OBS, PLT |

**Notes:**

- [socket2](https://crates.io/crates/socket2) for QUIC UDP socket tuning (buffer sizes, DF bit,
  platform-specific options), ICMP raw sockets
- [sysinfo](https://crates.io/crates/sysinfo) for diagnostic collection (CPU, memory, disk, network —
  replaces Go's diagnostic system collectors)
- [open](https://crates.io/crates/open) for platform-specific browser launch (access token flows):
  `xdg-open` (Linux), [open](https://crates.io/crates/open) (macOS), `cmd /c start` (Windows)
- [fd-lock](https://crates.io/crates/fd-lock) for token file locking and credential exclusive access
- [which](https://crates.io/crates/which) for finding executables (`traceroute`/`tracert` for
  diagnostics)
- [hostname](https://crates.io/crates/hostname) for hostname detection in management/diagnostics

---

## Layer 10 — Testing (Dev Dependencies)

### 10.1 Test Frameworks

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [proptest](https://crates.io/crates/proptest) | 1.x | 5 | 5 | 5 | Testing | TST (8 fuzz targets) |
| [mockall](https://crates.io/crates/mockall) | 0.13.x | 4 | 4 | 4 | Testing | TST |
| [rstest](https://crates.io/crates/rstest) | 0.25.x | 4 | 4 | 4 | Testing | TST |
| [test-case](https://crates.io/crates/test-case) | 3.x | 4 | 4 | 4 | Testing | TST |
| [pretty_assertions](https://crates.io/crates/pretty_assertions) | 1.x | 4 | 5 | 5 | Testing | TST |
| [assert_matches](https://crates.io/crates/assert_matches) | 1.x | 5 | 5 | 5 | Testing | TST |
| [insta](https://crates.io/crates/insta) | 1.x | 4 | 4 | 4 | Testing | TST |
| [claims](https://crates.io/crates/claims) | 0.7.x | 3 | 4 | 4 | Testing | TST |

**Notes:**

- [proptest](https://crates.io/crates/proptest) replaces Go fuzz targets (8 wire-protocol and session
  handling targets)
- [mockall](https://crates.io/crates/mockall) replaces `gomock` for interface mocking; S1 has ~80
  implicit interface satisfaction sites → ~80 traits to potentially
  mock
- [rstest](https://crates.io/crates/rstest) / `test-case` for table-driven tests (65% of Go tests are
  table-driven)
- [insta](https://crates.io/crates/insta) for snapshot testing — useful for wire format regression
  (datagram frame encoding, HTTP header serialization)
- [pretty_assertions](https://crates.io/crates/pretty_assertions) for readable diff output on test failures
- `claims` for assertion helpers (`assert_ok!`, `assert_err!`, etc.)

### 10.2 Benchmarking

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [criterion](https://crates.io/crates/criterion) | 0.5.x | 5 | 5 | 5 | Testing | TST (3 benchmarks) |
| [divan](https://crates.io/crates/divan) | 0.1.x | 4 | 3 | 4 | Testing | TST |

**Notes:**

- [criterion](https://crates.io/crates/criterion) for statistical benchmarks (3 Go benchmarks to port)
- `divan` is a lighter alternative with attribute macros

### 10.3 Test Utilities

| Crate | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [tracing-test](https://crates.io/crates/tracing-test) | 0.2.x | 3 | 3 | 4 | Testing | TST |
| [tokio-test](https://crates.io/crates/tokio-test) | 0.4.x | 5 | 5 | 5 | Testing | TST |
| [tempfile](https://crates.io/crates/tempfile) | 3.x | 5 | 5 | 5 | Testing | TST |
| [wiremock](https://crates.io/crates/wiremock) | 0.6.x | 4 | 4 | 4 | Testing | TST, UAC |
| [portpicker](https://crates.io/crates/portpicker) | 0.1.x | 3 | 4 | 4 | Testing | TST |

**Notes:**

- `tokio::io::duplex()` replaces `net.Pipe()` — no extra crate needed
- [tempfile](https://crates.io/crates/tempfile) for config file and credential file tests
- [wiremock](https://crates.io/crates/wiremock) for HTTP API mocking (cfapi client tests)
- [portpicker](https://crates.io/crates/portpicker) for dynamic port allocation in integration tests

---

## Layer 11 — Tooling and Guardrails

Tooling candidates used to enforce quality gates, dependency hygiene,
and policy checks during S2/S3/S4.

### 11.1 Dependency Debt and Policy Tooling

| Tool | Source | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [debtmap-cli](https://github.com/cloudflare/debtmap) | GitHub | 3 | 3 | 4 | Testing | TST, PF, IT |
| [debtmap (library)](https://github.com/cloudflare/debtmap) | GitHub | 3 | 3 | 4 | Facade | TST, PF, FEA |

**Notes:**

- [debtmap](https://crates.io/crates/debtmap) is an **internal/custom tool — not a public crates.io
  crate**. Source is the cloudflare/debtmap GitHub repository
- [debtmap](https://crates.io/crates/debtmap) belongs in this shopping cart for guardrails even if it is
  not linked into runtime binaries
- CLI mode supports CI checks and policy enforcement; library mode is
  useful for custom analysis hooks in repo-specific tooling
- Fits S2/S3 goals around controlled dependency expansion and
  explicit risk visibility

### 11.2 Supply Chain and Audit Tooling

| Tool | Version | T | M | S | Dimension | Span |
| --- | --- | --- | --- | --- | --- | --- |
| [cargo-deny](https://crates.io/crates/cargo-deny) | 0.16.x | 5 | 5 | 5 | Testing | TST, PF, DEP |
| [cargo-audit](https://crates.io/crates/cargo-audit) | 0.21.x | 5 | 5 | 5 | Testing | TST, PF, DEP |

**Notes:**

- `cargo-deny` enforces license policy, duplicate detection, source
  restrictions, and advisory database checks — runs in CI as a gate
- `cargo-audit` checks `Cargo.lock` against RustSec advisory database
  for known vulnerabilities
- Both are CI-only tools, not linked into runtime binaries

## Build-vs-Buy Scoring Model

This model compares integrating a crate versus writing/maintaining our
own implementation. Use it for every high-impact component.

### Scoring Dimensions (1–5)

| Dimension | Buy (Integrate Crate) | Build (Write Ourselves) |
| --- | --- | --- |
| Capability Fit | protocol/feature coverage now | exact semantic parity possible |
| Integration Cost | glue code and migration effort | implementation + maintenance effort |
| Operational Risk | CVEs, upstream churn, ecosystem health | bug surface, on-call burden |
| Performance Headroom | expected throughput/latency fit | ability to optimize for exact workload |
| Cognitive Load | complexity introduced to reviewers/devs | complexity of custom code ownership |
| Portability | cross-platform support quality | per-OS engineering tax |
| Exit Flexibility | ease of future replacement | lock-in to internal design choices |

### Weighted Decision Formula

Use weighted scores to avoid intuition-only decisions:

$$
Score(option)=\sum_{i=1}^{n} w_i \times s_i
$$

where $w_i$ is dimension weight and $s_i$ is 1–5 score.

### Suggested Weight Profiles

| Profile | Capability | Integration | Ops Risk | Performance | Cognitive | Portability | Exit |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Throughput-first | 0.20 | 0.10 | 0.15 | 0.30 | 0.10 | 0.10 | 0.05 |
| Maintainability-first | 0.15 | 0.20 | 0.20 | 0.10 | 0.20 | 0.10 | 0.05 |
| Portability-first | 0.15 | 0.10 | 0.15 | 0.10 | 0.15 | 0.25 | 0.10 |

### Decision Gates

| Gate | Rule | Action |
| --- | --- | --- |
| Adopt Crate | Buy score exceeds Build by >=0.5 | integrate with abstraction if needed |
| Build Ourselves | Build score exceeds Buy by >=0.5 | implement custom module with parity TOML first |
| Hedge | absolute difference <0.5 | keep behind facade and decide after spike |

### Initial High-Impact Scoring Backlog

| Component | Buy Candidate | Build Candidate | Why It Matters |
| --- | --- | --- | --- |
| QUIC transport | [quinn](https://crates.io/crates/quinn) or [quiche](https://crates.io/crates/quiche) | custom transport layer | highest architecture blast radius |
| Session manager | dashmap + tokio/ractor | custom v2/v3 manager | synchronization and migration semantics |
| ICMP handling | surge-ping + nix/socket2 | custom per-OS ICMP stack | heavy platform divergence |
| Logging pipeline | [tracing](https://crates.io/crates/tracing) stack + journald | custom compat logger | parity output + ops integration |
| CLI compatibility | [clap](https://crates.io/crates/clap) | custom compat layer over [clap](https://crates.io/crates/clap) | S4 behavioral parity expectations |
| Actor/supervisor | [ractor](https://crates.io/crates/ractor) | custom service registry | supervision tree design lock-in |
| Wire format parser | [nom](https://crates.io/crates/nom) | manual bytes slicing | custom binary protocol, exact control needed |
| HTTP management | [axum](https://crates.io/crates/axum) | raw [hyper](https://crates.io/crates/hyper) handlers | complexity vs simplicity tradeoff |
| Retry/backoff | [backon](https://crates.io/crates/backon) | custom backoff handler | Go has custom jitter/reset logic |
| Auto-updater | self_update | custom per-platform updater | platform divergence + update suppression |

### Applied Scorecards (Maintainability-First Profile)

Using weights: Capability 0.15, Integration 0.20, Ops Risk 0.20,
Performance 0.10, Cognitive 0.20, Portability 0.10, Exit 0.05.

#### QUIC Transport

| Dimension | Buy: [quinn](https://crates.io/crates/quinn) | Buy: [quiche](https://crates.io/crates/quiche) | Build: custom |
| --- | --- | --- | --- |
| Capability Fit | 4 (datagrams, streams, ALPN) | 5 (datagrams, FIPS, PQ native) | 5 (exact semantics) |
| Integration Cost | 5 (tokio-native, pure Rust) | 4 (C FFI, tokio-quiche Cloudflare-maintained) | 1 (massive effort) |
| Ops Risk | 4 (active community, no FIPS) | 4 (Cloudflare-maintained) | 2 (fully on-call owned) |
| Performance | 4 (no GSO/GRO yet) | 5 (production-proven at scale) | 3 (unproven) |
| Cognitive Load | 4 (well-documented API) | 3 (C FFI + async bridge) | 1 (custom protocol stack) |
| Portability | 5 (all platforms) | 3 (C build deps, FIPS Linux-primary) | 2 (per-platform work) |
| Exit Flexibility | 4 (transport facade viable) | 3 (facade viable but FFI baggage) | 2 (locked to internal design) |

**Weighted scores:** [quinn](https://crates.io/crates/quinn) = **4.20**, [quiche](https://crates.io/crates/quiche) = **3.55**, custom = **1.90**
**Decision gate:** Adopt [quinn](https://crates.io/crates/quinn) (Buy > Build by 2.30); hedge with
transport facade trait to enable quiche swap for FIPS path.

#### Session Manager (v2/v3)

| Dimension | Buy: dashmap + [ractor](https://crates.io/crates/ractor) | Build: custom |
| --- | --- | --- |
| Capability Fit | 3 (generic; migration/muxer logic is custom) | 5 (exact v2/v3 semantics) |
| Integration Cost | 4 (dashmap easy; [ractor](https://crates.io/crates/ractor) learning curve) | 3 (moderate; well-scoped ~500 lines) |
| Ops Risk | 3 (ractor pre-1.0) | 3 (owned bug surface) |
| Performance | 4 (dashmap sharding is fast) | 4 (can optimize for exact access pattern) |
| Cognitive Load | 3 (framework concepts to learn) | 4 (team owns the mental model) |
| Portability | 5 (pure Rust) | 5 (pure Rust) |
| Exit Flexibility | 3 (ractor shapes actor interfaces) | 4 (internal, no external lock-in) |

**Weighted scores:** Buy = **3.40**, Build = **3.85**
**Decision gate:** Build ourselves (Build > Buy by 0.45, borderline).
Use [dashmap](https://crates.io/crates/dashmap) for the registry but write the event loop, migration,
and muxer logic as custom code. Keep actor framework optional.

#### ICMP Handling

| Dimension | Buy: surge-ping + [nix](https://crates.io/crates/nix) | Build: custom per-OS |
| --- | --- | --- |
| Capability Fit | 3 (surge-ping is Linux-centric; no Windows ICMP API) | 5 (match all 3 OS paths exactly) |
| Integration Cost | 3 (still need Windows custom) | 3 (600 lines; well-understood protocol) |
| Ops Risk | 3 (surge-ping pre-1.0, single maintainer) | 3 (owned surface) |
| Performance | 4 (adequate for ICMP volume) | 4 (adequate) |
| Cognitive Load | 3 (two approaches: crate + custom Windows) | 3 (single consistent approach) |
| Portability | 2 (surge-ping lacks Windows; need split) | 4 (unified custom for all platforms) |
| Exit Flexibility | 3 (crate easy to replace) | 4 (no external dependency) |

**Weighted scores:** Buy = **2.95**, Build = **3.60**
**Decision gate:** Build ourselves (Build > Buy by 0.65). Use
[nix](https://crates.io/crates/nix)/[socket2](https://crates.io/crates/socket2) for raw socket plumbing but write ICMP router
logic per-platform.

#### Logging Pipeline

| Dimension | Buy: [tracing](https://crates.io/crates/tracing) + journald | Build: custom compat |
| --- | --- | --- |
| Capability Fit | 5 (structured spans, async-aware, ecosystem) | 4 (zerolog JSON exact match) |
| Integration Cost | 4 (well-integrated with [tokio](https://crates.io/crates/tokio) ecosystem) | 3 (custom Layer + JSON formatter) |
| Ops Risk | 5 (tracing is Rust standard) | 3 (maintenance burden for compat) |
| Performance | 4 (negligible overhead) | 4 (negligible) |
| Cognitive Load | 4 (well-known by Rust developers) | 3 (custom code to learn) |
| Portability | 5 (cross-platform) | 5 (cross-platform) |
| Exit Flexibility | 4 (tracing is de facto standard) | 3 (locked to custom format) |

**Weighted scores:** Buy = **4.45**, Build = **3.40**
**Decision gate:** Adopt [tracing](https://crates.io/crates/tracing) stack (Buy > Build by 1.05).
Build a thin `logging-compat` feature flag that adds zerolog JSON
output as a custom [tracing-subscriber](https://crates.io/crates/tracing-subscriber) Layer.

#### Actor/Supervisor Framework

| Dimension | Buy: [ractor](https://crates.io/crates/ractor) | Build: custom registry |
| --- | --- | --- |
| Capability Fit | 4 (typed actors, supervision trees) | 4 (exact Go overwatch semantics) |
| Integration Cost | 3 (ractor API learning curve) | 4 (Go overwatch is ~300 lines) |
| Ops Risk | 2 (pre-1.0, smaller community) | 3 (owned, small surface) |
| Performance | 4 (mailbox overhead acceptable) | 4 (can use channels directly) |
| Cognitive Load | 3 (actor framework concepts) | 4 (simpler mental model) |
| Portability | 5 (pure Rust) | 5 (pure Rust) |
| Exit Flexibility | 2 (framework shapes all service boundaries) | 4 (trait-based, swappable) |

**Weighted scores:** Buy = **3.15**, Build = **3.85**
**Decision gate:** Build ourselves (Build > Buy by 0.70). Write a
custom trait-based service registry matching Go's overwatch. Use
[crossbeam-channel](https://crates.io/crates/crossbeam-channel) or `tokio::mpsc` for messaging. Keep [ractor](https://crates.io/crates/ractor) as a
future option behind facade.

---

## Rewrite-Ourselves Candidates

Areas where custom implementation is likely preferable.

| Domain | Go Lines | Crate Alternative | Rationale |
| --- | --- | --- | --- |
| SOCKS5 proxy | ~400 | [fast-socks5](https://crates.io/crates/fast-socks5) (pre-1.0, T:3) | Small protocol; Go impl is minimal |
| ICMP proxy | ~600 | [surge-ping](https://crates.io/crates/surge-ping) (pre-1.0, T:3) | Platform-divergent; need full raw-socket control |
| Bidirectional stream relay | ~100 | `tokio::io::copy_bidirectional` | stdlib equivalent in [tokio](https://crates.io/crates/tokio) |
| Backoff handler | ~200 | [backon](https://crates.io/crates/backon) (T:4) | Go has custom jitter; evaluate coverage |
| Session manager (v2/v3) | ~500 | — | Custom concurrent registry with dashmap + [tokio](https://crates.io/crates/tokio) |
| Fuse/latch primitives | ~50 | — | Trivial with `tokio::sync::Notify` or `watch` |
| Overwatch service manager | ~300 | [ractor](https://crates.io/crates/ractor) (T:3) | Go impl simple; actor framework may be overkill |
| Auto-updater | ~400 | `self_update` (T:3) | Platform-specific logic; update suppression rules |
| FNV-32a feature selector | ~30 | [fnv](https://crates.io/crates/fnv) (T:5) | Trivial, but [fnv](https://crates.io/crates/fnv) crate is reliable |
| Datagram v2/v3 wire format | ~300 | [nom](https://crates.io/crates/nom) or manual [bytes](https://crates.io/crates/bytes) | Custom binary format needs custom parser |
| Config precedence merge | ~200 | — | 5-layer merge logic is app-specific |
| Flow/limiter | ~200 | [governor](https://crates.io/crates/governor) (T:4) | Token-bucket rate limiting; evaluate governor coverage |
| Edge address pool | ~300 | — | Region/priority/rotation logic is app-specific |
| ConnAwareLogger | ~50 | — | Custom [tracing](https://crates.io/crates/tracing) `Layer` implementation |
| CLI compat wrapper | ~500 | [clap](https://crates.io/crates/clap) + custom | If `cli-compat` flag needed, wraps [clap](https://crates.io/crates/clap) |
| Logging compat facade | ~300 | [tracing](https://crates.io/crates/tracing) + custom | If `logging-compat` flag needed, zerolog JSON |
| cfapi envelope client | ~400 | [reqwest](https://crates.io/crates/reqwest) + custom | Envelope pattern, pagination, error aggregation |
| Diagnostic artifact bundle | ~200 | [zip](https://crates.io/crates/zip) + [sysinfo](https://crates.io/crates/sysinfo) | Collection logic is app-specific |

### Custom Facade Opportunities

These are areas where building our own abstraction layer enables
feature-flagged alternatives:

#### 1. Logging Facade

- Abstraction over [tracing](https://crates.io/crates/tracing) (native) and zerolog-compatible JSON
  (compat)
- Upstream management WebSocket log streaming needs a custom
  [tracing-subscriber](https://crates.io/crates/tracing-subscriber) `Layer` regardless
- Feature flag: `logging-compat` emits zerolog-compatible JSON,
  `logging-native` emits rich [tracing](https://crates.io/crates/tracing) spans

#### 2. CLI Facade

- Abstraction over [clap](https://crates.io/crates/clap) that can output `urfave/cli`-cadence help
  text and error messages for behavioral parity testing in S4
- Feature flag: `cli-compat` matches Go output exactly,
  `cli-native` uses idiomatic [clap](https://crates.io/crates/clap)

#### 3. Transport Facade

- Abstraction over quinn/quiche QUIC backends
- Feature flag: `quic-quinn` vs `quic-quiche`
- Trait: `QuicConnection`, `QuicStream`, `DatagramSender/Receiver`

---

## Catalog → Dependency Cross-Reference

Complete mapping from every S1 catalog to the dependencies that
cover it.

### Domain Catalogs (22)

| Catalog | Primary Dependencies | Layers |
| --- | --- | --- |
| access-policies | [jsonwebtoken](https://crates.io/crates/jsonwebtoken), [ipnet](https://crates.io/crates/ipnet), [reqwest](https://crates.io/crates/reqwest), [crypto_box](https://crates.io/crates/crypto_box), [x25519-dalek](https://crates.io/crates/x25519-dalek), [fd-lock](https://crates.io/crates/fd-lock), [open](https://crates.io/crates/open) | 7, 8 |
| capnp-rpc | [capnp](https://crates.io/crates/capnp), [capnpc](https://crates.io/crates/capnpc), [capnp-rpc](https://crates.io/crates/capnp-rpc), [tokio](https://crates.io/crates/tokio)/[crossbeam](https://crates.io/crates/crossbeam) | 2, 1 |
| cli | [clap](https://crates.io/crates/clap), [serde](https://crates.io/crates/serde), [serde_yaml](https://crates.io/crates/serde_yaml), [chrono](https://crates.io/crates/chrono)/[time](https://crates.io/crates/time) | 6, 2 |
| config | [serde](https://crates.io/crates/serde), [serde_yaml](https://crates.io/crates/serde_yaml), [toml](https://crates.io/crates/toml), [serde_json](https://crates.io/crates/serde_json), [notify](https://crates.io/crates/notify), [clap](https://crates.io/crates/clap), [tokio](https://crates.io/crates/tokio), [sha2](https://crates.io/crates/sha2) | 2, 6, 7 |
| const-and-env | (stdlib `const`, `std::env`, [once_cell](https://crates.io/crates/once_cell)) | 1 |
| crypto | [rustls](https://crates.io/crates/rustls)/[boring](https://crates.io/crates/boring), [ring](https://crates.io/crates/ring), [sha2](https://crates.io/crates/sha2), [hmac](https://crates.io/crates/hmac), [p256](https://crates.io/crates/p256), [ecdsa](https://crates.io/crates/ecdsa), [pem](https://crates.io/crates/pem), `x509-parser`, [ssh-key](https://crates.io/crates/ssh-key), [webpki-roots](https://crates.io/crates/webpki-roots), `rand` | 3, 8 |
| deployments | [clap](https://crates.io/crates/clap), [dirs](https://crates.io/crates/dirs), [reqwest](https://crates.io/crates/reqwest), [plist](https://crates.io/crates/plist), [sd-notify](https://crates.io/crates/sd-notify)/[systemd](https://crates.io/crates/systemd), [windows-service](https://crates.io/crates/windows-service), [flate2](https://crates.io/crates/flate2), `self_update` | 6, 7, 9 |
| edge-interactions | [hickory-resolver](https://crates.io/crates/hickory-resolver), [quinn](https://crates.io/crates/quinn)/[quiche](https://crates.io/crates/quiche), [hyper](https://crates.io/crates/hyper)/[h2](https://crates.io/crates/h2), [tokio-tungstenite](https://crates.io/crates/tokio-tungstenite), [reqwest](https://crates.io/crates/reqwest), [capnp-rpc](https://crates.io/crates/capnp-rpc) | 3, 7 |
| host-interactions | [notify](https://crates.io/crates/notify), [nix](https://crates.io/crates/nix), [socket2](https://crates.io/crates/socket2), [fd-lock](https://crates.io/crates/fd-lock), [sysinfo](https://crates.io/crates/sysinfo), [tokio](https://crates.io/crates/tokio)::process | 7, 9 |
| ingress | [hyper](https://crates.io/crates/hyper), `tokio::net`, [tokio-tungstenite](https://crates.io/crates/tokio-tungstenite), [hickory-resolver](https://crates.io/crates/hickory-resolver), [socket2](https://crates.io/crates/socket2), [regex](https://crates.io/crates/regex), [ipnet](https://crates.io/crates/ipnet), `url` | 3, 7 |
| metrics | [prometheus-client](https://crates.io/crates/prometheus-client)/[metrics](https://crates.io/crates/metrics), [hyper](https://crates.io/crates/hyper)/[axum](https://crates.io/crates/axum) | 4, 7 |
| observabilities | [tracing](https://crates.io/crates/tracing), [tracing-subscriber](https://crates.io/crates/tracing-subscriber), [tracing-journald](https://crates.io/crates/tracing-journald), [opentelemetry](https://crates.io/crates/opentelemetry)-\*, [serde_json](https://crates.io/crates/serde_json), `sentry`, `foundations` | 4, 7 |
| overwatch | [tokio](https://crates.io/crates/tokio) / [ractor](https://crates.io/crates/ractor), custom service registry | 1, 7 |
| platforms | [nix](https://crates.io/crates/nix), [windows-service](https://crates.io/crates/windows-service), [windows-sys](https://crates.io/crates/windows-sys), [socket2](https://crates.io/crates/socket2), [sd-notify](https://crates.io/crates/sd-notify)/[systemd](https://crates.io/crates/systemd), [plist](https://crates.io/crates/plist), [open](https://crates.io/crates/open), [sysinfo](https://crates.io/crates/sysinfo) | 9 |
| proxying | [hyper](https://crates.io/crates/hyper)/[h2](https://crates.io/crates/h2), [quinn](https://crates.io/crates/quinn)/[quiche](https://crates.io/crates/quiche), [tokio-tungstenite](https://crates.io/crates/tokio-tungstenite), `tokio::io`, [bytes](https://crates.io/crates/bytes), custom SOCKS5/ICMP | 3, 7 |
| sessions | [quinn](https://crates.io/crates/quinn)/[quiche](https://crates.io/crates/quiche) (datagram API), [tokio](https://crates.io/crates/tokio), [uuid](https://crates.io/crates/uuid), [dashmap](https://crates.io/crates/dashmap), [parking_lot](https://crates.io/crates/parking_lot), [nom](https://crates.io/crates/nom) | 3, 1, 2 |
| shared-state | [tokio](https://crates.io/crates/tokio)::sync / [crossbeam-channel](https://crates.io/crates/crossbeam-channel), [dashmap](https://crates.io/crates/dashmap), [arc-swap](https://crates.io/crates/arc-swap), [parking_lot](https://crates.io/crates/parking_lot) | 1 |
| state-machines | (enum-based FSM — stdlib), [tokio](https://crates.io/crates/tokio), [backon](https://crates.io/crates/backon) | 1, 7 |
| supervisor | [tokio](https://crates.io/crates/tokio) JoinSet / [ractor](https://crates.io/crates/ractor), [backon](https://crates.io/crates/backon), [tracing](https://crates.io/crates/tracing), cancellation tokens | 1, 7 |
| tunnels | ALL transport crates, [reqwest](https://crates.io/crates/reqwest), [capnp-rpc](https://crates.io/crates/capnp-rpc), [tokio](https://crates.io/crates/tokio), [clap](https://crates.io/crates/clap) | 3, 7 (integration) |
| tunnels-transport | [quinn](https://crates.io/crates/quinn)/[quiche](https://crates.io/crates/quiche), [hyper](https://crates.io/crates/hyper)/[h2](https://crates.io/crates/h2), [boring](https://crates.io/crates/boring)/[rustls](https://crates.io/crates/rustls), [bytes](https://crates.io/crates/bytes), [base64](https://crates.io/crates/base64) | 3, 2 |
| upstream-api-contracts | [reqwest](https://crates.io/crates/reqwest), [serde](https://crates.io/crates/serde)/[serde_json](https://crates.io/crates/serde_json), [axum](https://crates.io/crates/axum)/[hyper](https://crates.io/crates/hyper), [tokio-tungstenite](https://crates.io/crates/tokio-tungstenite), `url` | 3, 7 |

### Cross-Cutting Catalogs (8)

| Catalog | Primary Dependencies | Layers |
| --- | --- | --- |
| concurrency | [tokio](https://crates.io/crates/tokio) / [crossbeam-channel](https://crates.io/crates/crossbeam-channel) / [parking_lot](https://crates.io/crates/parking_lot), [dashmap](https://crates.io/crates/dashmap), [arc-swap](https://crates.io/crates/arc-swap), [futures](https://crates.io/crates/futures) | 1 |
| error-propagation | [thiserror](https://crates.io/crates/thiserror), [anyhow](https://crates.io/crates/anyhow), [tracing](https://crates.io/crates/tracing), `sentry` | 4, 5 |
| features | [clap](https://crates.io/crates/clap), [hickory-resolver](https://crates.io/crates/hickory-resolver), [jsonwebtoken](https://crates.io/crates/jsonwebtoken), [reqwest](https://crates.io/crates/reqwest), [serde](https://crates.io/crates/serde), [tokio-tungstenite](https://crates.io/crates/tokio-tungstenite), [fnv](https://crates.io/crates/fnv) | 6, 7, 8 |
| init-teardown | cancellation tokens, `tokio::signal` / [signal-hook](https://crates.io/crates/signal-hook), [sd-notify](https://crates.io/crates/sd-notify), [tracing](https://crates.io/crates/tracing) — note: `signal/safe_signal` maps to `tokio::sync::Notify` or `watch` (no external dep) | 1, 9 |
| platform-substrates | [nix](https://crates.io/crates/nix), [windows-sys](https://crates.io/crates/windows-sys), [socket2](https://crates.io/crates/socket2), [boring](https://crates.io/crates/boring) (FIPS), platform service crates | 9, 3 |
| porting-friction | [thiserror](https://crates.io/crates/thiserror), [bytes](https://crates.io/crates/bytes), [bstr](https://crates.io/crates/bstr), [once_cell](https://crates.io/crates/once_cell), [async-trait](https://crates.io/crates/async-trait), [smallvec](https://crates.io/crates/smallvec) | 1, 2, 5 |
| wire-protocol | [quinn](https://crates.io/crates/quinn)/[quiche](https://crates.io/crates/quiche), [h2](https://crates.io/crates/h2)/[hyper](https://crates.io/crates/hyper), [capnp](https://crates.io/crates/capnp), [rustls](https://crates.io/crates/rustls)/[boring](https://crates.io/crates/boring), [bytes](https://crates.io/crates/bytes), [base64](https://crates.io/crates/base64), [prost](https://crates.io/crates/prost), [nom](https://crates.io/crates/nom) | 2, 3 |
| tests | [proptest](https://crates.io/crates/proptest), [mockall](https://crates.io/crates/mockall), [rstest](https://crates.io/crates/rstest), [criterion](https://crates.io/crates/criterion), [tracing-test](https://crates.io/crates/tracing-test), [tokio-test](https://crates.io/crates/tokio-test), [wiremock](https://crates.io/crates/wiremock), [tempfile](https://crates.io/crates/tempfile), [insta](https://crates.io/crates/insta), [debtmap-cli](https://crates.io/crates/debtmap-cli) | 10, 11 |

---

## Platform Coverage Matrix

Crate-level platform support for Linux, macOS, and Windows.
Legend: ✅ full support, ⚠️ partial/degraded, ❌ not available.

| Crate | Linux | macOS | Windows | Notes |
| --- | --- | --- | --- | --- |
| [nix](https://crates.io/crates/nix) | ✅ | ✅ | ❌ | Unix-only; Windows uses [windows-sys](https://crates.io/crates/windows-sys) |
| [windows-service](https://crates.io/crates/windows-service) | ❌ | ❌ | ✅ | Windows service lifecycle |
| [windows-sys](https://crates.io/crates/windows-sys) | ❌ | ❌ | ✅ | Low-level Windows API |
| [sd-notify](https://crates.io/crates/sd-notify) / [systemd](https://crates.io/crates/systemd) | ✅ | ❌ | ❌ | Linux/systemd only |
| [tracing-journald](https://crates.io/crates/tracing-journald) | ✅ | ❌ | ❌ | Linux/systemd only |
| [plist](https://crates.io/crates/plist) | ❌ | ✅ | ❌ | macOS launchd plist |
| [boring](https://crates.io/crates/boring) / `boring-sys` | ✅ | ✅ | ⚠️ | Windows: MSVC build chain required |
| [socket2](https://crates.io/crates/socket2) | ✅ | ✅ | ✅ | Full cross-platform |
| [sysinfo](https://crates.io/crates/sysinfo) | ✅ | ✅ | ✅ | Full cross-platform |
| `rustls-platform-verifier` | ✅ | ✅ | ✅ | Delegates to OS verifier |
| [glommio](https://crates.io/crates/glommio) | ✅ | ❌ | ❌ | Linux io_uring only (pre-decided out) |

---

## ADR Candidates

Decisions requiring formal ADRs in phase 2.4 based on this analysis:

| # | Decision | Catalogs Affected | Risk |
| --- | --- | --- | --- |
| 1 | [quinn](https://crates.io/crates/quinn) vs [quiche](https://crates.io/crates/quiche) for QUIC transport | TUN, PRX, TT, SES, EDG, WP, SM, CON | Critical |
| 2 | FIPS + QUIC integration strategy | CRY, TT, PS, WP | Critical |
| 3 | Actor framework vs custom supervisor | OVW, SUP, SES | High |
| 4 | tower-based middleware vs flat dispatch | PRX, ING, MET, OBS | Medium |
| 5 | [prometheus-client](https://crates.io/crates/prometheus-client) vs [metrics](https://crates.io/crates/metrics) facade | MET + all instrumented modules | Medium |
| 6 | [axum](https://crates.io/crates/axum) vs raw [hyper](https://crates.io/crates/hyper) for management | UAC, MET, OBS | Low |
| 7 | SOCKS5: [fast-socks5](https://crates.io/crates/fast-socks5) vs custom | PRX | Low |
| 8 | ICMP: [surge-ping](https://crates.io/crates/surge-ping) vs custom raw socket | ING, PRX, PLT, PS | Medium |
| 9 | Overwatch: [ractor](https://crates.io/crates/ractor) vs custom service registry | OVW, SUP | Medium |
| 10 | CLI: [clap](https://crates.io/crates/clap) native vs compat wrapper | CLI, CFG | Medium |
| 11 | Logging: native [tracing](https://crates.io/crates/tracing) vs zerolog-compat facade | OBS, ALL modules | Medium |
| 12 | RustCrypto vs [ring](https://crates.io/crates/ring) primitives | CRY | Low |
| 13 | Non-async event loops: [crossbeam](https://crates.io/crates/crossbeam) for v2 session mgr | SES, CON | Medium |
| 14 | Auto-updater: `self_update` vs custom | DEP | Low |
| 15 | Adopt `foundations` selectively vs fully custom bootstrap | OBS, CON, EP, DEP | Medium |
| 16 | Keep `pingora` as selective dependency vs no adoption | PRX, TT, ING | Low |
| 17 | Wire format parsing: [nom](https://crates.io/crates/nom) vs manual [bytes](https://crates.io/crates/bytes) slicing | WP, SES, TT | Medium |
| 18 | Runtime debugging: `tokio-console` feature gate strategy | CON, OBS | Low |

**Pre-decided (not requiring ADR):** [tokio](https://crates.io/crates/tokio) runtime (§1.1),
[serde_yaml](https://crates.io/crates/serde_yaml) 0.9.x (§2.1), [chrono](https://crates.io/crates/chrono) 0.4.x (§6.2), MSRV ≥ 1.75
(§1.2), h3 deferred to FC (§3.1), mimalloc feature-flagged (§1.6).

---

## lib.rs Ecosystem Classification

Crate categorization sourced from [lib.rs](https://lib.rs), the
community-maintained Rust crate directory. lib.rs applies manual
editorial categorization (unlike crates.io's author-chosen keywords),
providing a curated ecosystem taxonomy. Rank is the crate's position
within its primary lib.rs category. Downloads are monthly.

### Purpose

This dimension complements the existing T/M/S scoring and shopping cart
layers. lib.rs categories reveal:

- **Ecosystem dominance** — crates ranked #1–3 in their category are
  de facto standards
- **Category concentration** — when multiple shopping cart crates share
  a lib.rs category, they compete for the same niche
- **Maturity signal** — high download counts with top-3 rank indicate
  strong ecosystem convergence

### lib.rs Category Map

| Crate | lib.rs Category | Rank | Downloads/mo | Layer | Key Findings |
| --- | --- | --- | --- | --- | --- |
| [tokio](https://crates.io/crates/tokio) | Asynchronous | #20 | 40.1M | 1 | v1.50.0, MSRV 1.71 |
| [tokio-util](https://crates.io/crates/tokio-util) | Asynchronous | #953 | 32.9M | 1 | |
| `tokio-stream` | Asynchronous | #150 | 22.6M | 1 | |
| [futures](https://crates.io/crates/futures) | Asynchronous | #1 | 34.4M | 1 | De facto standard for futures trait extensions |
| [futures-lite](https://crates.io/crates/futures-lite) | Asynchronous | #9 | 12.6M | 1 | Lighter alternative, smol ecosystem |
| [glommio](https://crates.io/crates/glommio) | Concurrency | #392 | 3.2K | 1 | Niche — thread-per-core, io-uring only |
| [crossbeam-channel](https://crates.io/crates/crossbeam-channel) | Concurrency | #107 | 22.5M | 1 | MPMC channels, Go-style select |
| [parking_lot](https://crates.io/crates/parking_lot) | Concurrency | #1 | 44.8M | 1 | **Category leader.** 1.5–5× faster than std Mutex |
| [dashmap](https://crates.io/crates/dashmap) | Concurrency | #10 | 17.8M | 1 | v7.0.0-rc2 upcoming, MSRV 1.70 |
| [signal-hook](https://crates.io/crates/signal-hook) | Rust patterns | #17 | 11.7M | 1 | v0.4.3, safe Unix signal handling |
| [governor](https://crates.io/crates/governor) | Network programming | #17 | 4.2M | 1 | GCRA rate limiter, 64-bit atomic state |
| `moka` | Caching | #1 | 6.9M | 1 | **Category leader.** Caffeine-inspired, TinyLFU |
| [serde](https://crates.io/crates/serde) | Encoding | #42 | 50M | 2 | v1.0.228 |
| [serde_json](https://crates.io/crates/serde_json) | Encoding | #1 | 50.3M | 2 | **Category leader.** 500–1000 MB/s deser |
| [serde_yaml](https://crates.io/crates/serde_yaml) | Encoding | #2981 | 18.7M | 2 | **Deprecated upstream.** Still 18.7M downloads/mo |
| [capnp](https://crates.io/crates/capnp) | Memory management | #10 | 797K | 2 | v0.25.2 (cart lists 0.20.x — **update needed**) |
| [prost](https://crates.io/crates/prost) | Encoding | #3 | 29M | 2 | **Passively maintained.** Expects official `protobuf` crate to supersede |
| [bytes](https://crates.io/crates/bytes) | Rust patterns | #23 | 43.7M | 2 | v1.11.1, tokio-rs org |
| [uuid](https://crates.io/crates/uuid) | Value formatting | #1 | 34.1M | 2 | **Category leader.** v1.22.0 |
| `url` | Parser implementations | #6 | 38.6M | 2 | v2.5.8, WHATWG URL Standard |
| [http](https://crates.io/crates/http) | Rust patterns | #6 | 44.9M | 2 | v1.4.0, hyperium org |
| [smallvec](https://crates.io/crates/smallvec) | Data structures | #4 | 47.7M | 2 | v2.0.0-alpha.12 in development |
| [quinn](https://crates.io/crates/quinn) | Network programming | #7 | 17M | 3 | v0.11.9, has `rustls-aws-lc-rs-fips` feature |
| [quiche](https://crates.io/crates/quiche) | Network programming | #52 | 179K | 3 | v0.26.1, **Cloudflare-owned**, uses BoringSSL, MSRV 1.85 |
| `tokio-quiche` | Network programming | #1582 | 129K | 3 | v0.16.1, **Cloudflare-owned**, wraps [quiche](https://crates.io/crates/quiche)+[tokio](https://crates.io/crates/tokio) |
| [hyper](https://crates.io/crates/hyper) | Web programming | #1 (HTTP server) | 40.6M | 3 | **Category leader.** |
| [h2](https://crates.io/crates/h2) | Asynchronous | #2 | 36.1M | 3 | v0.4.13 |
| [rustls](https://crates.io/crates/rustls) | Cryptography | #2 | 43.9M | 3 | v0.24.0-dev.0, recommends `aws-lc-rs` as default provider |
| [reqwest](https://crates.io/crates/reqwest) | Web programming | #2 (HTTP client) | 32.5M | 3 | v0.13.x available (cart lists 0.12.x — **update needed**) |
| [axum](https://crates.io/crates/axum) | Web programming | #2 (HTTP server) | 22.4M | 3 | v0.8.8, cart lists 0.8.x (correct) |
| [tower](https://crates.io/crates/tower) | Network programming | #1 | 32.5M | 3 | **Category leader.** Service trait standard |
| `pingora` | Network programming | #87 | 781K | 3 | v0.8.0, **Cloudflare-owned**, MSRV 1.84 |
| [socket2](https://crates.io/crates/socket2) | Network programming | #6 | 57.8M | 3/9 | v0.6.3, rust-lang org, MSRV 1.70 |
| [tracing](https://crates.io/crates/tracing) | Debugging | #1 | 37M | 4 | **Category leader.** v0.1.44, MSRV 1.65 |
| [clap](https://crates.io/crates/clap) | Command-line interface | #7 | 43.9M | 6 | v4.6.0, Rust 2024 edition |
| [ipnet](https://crates.io/crates/ipnet) | Network programming | #4 | 24.7M | 7 | v2.12.0, IPv4/IPv6 prefix types |
| [ractor](https://crates.io/crates/ractor) | Asynchronous | #28 | 55K | 7 | v0.15.12, Erlang-inspired, used at Meta |
| `aws-lc-rs` | Cryptography | #20 | 13.6M | 8 | v1.16.2, **FIPS via `fips` feature**, PQ crypto support |
| `rustls-post-quantum` | Cryptography | #2382 | 6.7K | 8 | ML-KEM moved to [rustls](https://crates.io/crates/rustls) itself since v0.23.22 |
| [boring](https://crates.io/crates/boring) | Cryptography | #35 | 238K | 8 | v5.0.2, **Cloudflare-owned**, `fips`+`mlkem`+`rpk` features |
| [nix](https://crates.io/crates/nix) | Unix APIs | #3 | 37M | 9 | v0.31.2, MSRV 1.69 |
| [thiserror](https://crates.io/crates/thiserror) | Rust patterns | #6 | 68.2M | 5 | v2.0.18, MSRV 1.68 |
| [anyhow](https://crates.io/crates/anyhow) | Rust patterns | #3 | 41.6M | 5 | v1.0.102 |
| [miette](https://crates.io/crates/miette) | Rust patterns | #21 | 4.2M | 5 | v7.6.0, fancy diagnostics, MSRV 1.70 |
| [prometheus-client](https://crates.io/crates/prometheus-client) | Text processing | #26 | 2.2M | 4 | v0.24.0, Open Metrics spec |
| [proptest](https://crates.io/crates/proptest) | Testing | #8 | 9.3M | 10 | v1.10.0, MSRV 1.84, Hypothesis-inspired |
| [mockall](https://crates.io/crates/mockall) | Testing | #15 | 8M | 10 | v0.14.0, MSRV 1.77 |

### Key Findings from lib.rs Research

#### Version Updates Needed

| Crate | Cart Version | lib.rs Latest | Action |
| --- | --- | --- | --- |
| [capnp](https://crates.io/crates/capnp) | 0.20.x | 0.25.2 | Update — significant version gap |
| [reqwest](https://crates.io/crates/reqwest) | 0.12.x | 0.13.x | Evaluate — new major bump available |
| [boring](https://crates.io/crates/boring) | 4.x | 5.0.2 | Update — major breaking changes in v5 |

#### Ecosystem Intelligence

- **`aws-lc-rs` is now the recommended default crypto provider for
  [rustls](https://crates.io/crates/rustls)** — not just [ring](https://crates.io/crates/ring). Has native FIPS support via `fips`
  feature flag and post-quantum crypto support. This strengthens
  the [quinn](https://crates.io/crates/quinn)+[rustls](https://crates.io/crates/rustls)+[aws-lc-rs](https://crates.io/crates/aws-lc-rs) FIPS path as a viable alternative to
  [boring](https://crates.io/crates/boring)+[quiche](https://crates.io/crates/quiche)
- **`rustls-post-quantum`** — ML-KEM support has been moved into
  [rustls](https://crates.io/crates/rustls) itself since v0.23.22; use the `prefer-post-quantum`
  feature flag. The standalone crate is now mostly for ML-DSA via
  `aws-lc-rs-unstable`
- **`boring-rustls-provider`** does NOT exist on lib.rs — this was
  a hypothesis, not a real crate. Remove from consideration
- **[boring](https://crates.io/crates/boring) v5.0.2** has breaking changes from v4; adds `fips`,
  `mlkem`, `rpk` features; `hyper 1.x` only compatibility.
  Cloudflare-owned
- **[quiche](https://crates.io/crates/quiche) v0.26.1** has `boringssl-boring-crate` feature allowing
  use of the [boring](https://crates.io/crates/boring) crate for TLS — this simplifies the
  [boring](https://crates.io/crates/boring)+[quiche](https://crates.io/crates/quiche) integration story
- **`tokio-quiche` v0.16.1** is Cloudflare's official async wrapper
  around [quiche](https://crates.io/crates/quiche)+[boring](https://crates.io/crates/boring); depends on the `foundations` crate
- **[prost](https://crates.io/crates/prost) is passively maintained** — maintainer expects the
  official `protobuf` crate to supersede it. Factor this into the
  protobuf decision for datagram v2 tracing spans
- **[serde_yaml](https://crates.io/crates/serde_yaml) deprecated** but still at 18.7M downloads/mo —
  confirms the pre-decision to use 0.9.x in S3 with planned
  migration is sound
- **`pingora`** — Cloudflare-owned, v0.8.0, supports HTTP 1/2 proxy,
  TLS over OpenSSL/BoringSSL/rustls, MSRV 1.84. Ranks #87 in
  Network programming. Worth evaluating for selective reuse

#### Category Leaders Adopted

Crates ranked #1–3 in their lib.rs category that appear in this
shopping cart — these represent ecosystem convergence points:

| Crate | Category | Rank | Implication |
| --- | --- | --- | --- |
| [futures](https://crates.io/crates/futures) | Asynchronous | #1 | De facto async trait extensions |
| [parking_lot](https://crates.io/crates/parking_lot) | Concurrency | #1 | De facto sync primitives |
| `moka` | Caching | #1 | De facto async cache |
| [serde_json](https://crates.io/crates/serde_json) | Encoding | #1 | De facto JSON |
| [uuid](https://crates.io/crates/uuid) | Value formatting | #1 | De facto identifier |
| [hyper](https://crates.io/crates/hyper) | HTTP server | #1 | De facto HTTP |
| [tower](https://crates.io/crates/tower) | Network programming | #1 | De facto service middleware |
| [tracing](https://crates.io/crates/tracing) | Debugging | #1 | De facto instrumentation |
| [rustls](https://crates.io/crates/rustls) | Cryptography | #2 | De facto Rust-native TLS |
| [reqwest](https://crates.io/crates/reqwest) | HTTP client | #2 | De facto HTTP client |
| [axum](https://crates.io/crates/axum) | HTTP server | #2 | De facto web framework |
| [h2](https://crates.io/crates/h2) | Asynchronous | #2 | De facto HTTP/2 |
| [anyhow](https://crates.io/crates/anyhow) | Rust patterns | #3 | De facto app error handling |
| [nix](https://crates.io/crates/nix) | Unix APIs | #3 | De facto Unix syscalls |
| [prost](https://crates.io/crates/prost) | Encoding | #3 | De facto protobuf (but passively maintained) |

---

## Risk Summary

### Critical-Risk Dependencies (architectural lock-in)

| Dependency | Risk | Mitigation |
| --- | --- | --- |
| [quinn](https://crates.io/crates/quinn) / [quiche](https://crates.io/crates/quiche) | QUIC stack choice affects FIPS, PQ, datagram v2/v3 wire format, session lifecycle — touches 8+ catalogs | ADR-001 with prototype spike; transport facade trait |
| [boring](https://crates.io/crates/boring) / [rustls](https://crates.io/crates/rustls) | FIPS toggle must work at build time; PQ curve support varies (X25519MLKEM768 vs P256Kyber768Draft00) | Cargo feature split, CI matrix |
| [tokio](https://crates.io/crates/tokio) | Runtime choice colors entire dependency graph; pre-decided (§1.1) but ecosystem lock-in remains a structural risk | Runtime facade trait if benchmarks warrant [glommio](https://crates.io/crates/glommio) |
| [capnp-rpc](https://crates.io/crates/capnp-rpc) | Single ecosystem maintainer; control plane depends entirely on it | Evaluate schema compat early; fallback: gRPC (scope explosion) |

### High-Risk Dependencies (significant effort to substitute)

| Dependency | Risk | Mitigation |
| --- | --- | --- |
| [hyper](https://crates.io/crates/hyper) 1.x | Major API change from 0.14; utility ecosystem still stabilizing | Pin version, wrap in transport trait |
| [tower](https://crates.io/crates/tower) | If adopted deeply, shapes entire request pipeline | Limit to middleware composition; keep core logic trait-based |
| [ractor](https://crates.io/crates/ractor) / actor choice | Actor framework shapes supervision tree design | Prototype in S3/3.5; keep overwatch trait-based |

### Medium-Risk Dependencies (substitutable with moderate effort)

| Dependency | Risk | Mitigation |
| --- | --- | --- |
| `opentelemetry-*` | Pre-1.0, API churn | Isolate behind [tracing](https://crates.io/crates/tracing) layer |
| [serde_yaml](https://crates.io/crates/serde_yaml) / [serde_yml](https://crates.io/crates/serde_yml) | Upstream deprecation / version immaturity | ADR with migration path |
| [prometheus-client](https://crates.io/crates/prometheus-client) / [metrics](https://crates.io/crates/metrics) | API differences; metrics facade vs direct use | ADR; isolate behind trait |
| [hickory-resolver](https://crates.io/crates/hickory-resolver) | Pre-1.0, occasional API changes | Isolate behind DNS trait |

### Low-Risk Dependencies (easily substitutable)

All Layer 1 sync primitives ([parking_lot](https://crates.io/crates/parking_lot), `crossbeam-*`), all
Layer 2 serialization ([serde](https://crates.io/crates/serde), [serde_json](https://crates.io/crates/serde_json)), all Layer 10 testing
crates, most Layer 9 platform crates, most Layer 8 RustCrypto
crates, and most Layer 11 tooling crates.

---

## Dependency Count Summary

| Layer | Count | Examples |
| --- | --- | --- |
| 1 — Runtime/Concurrency | 25 | [tokio](https://crates.io/crates/tokio), [glommio](https://crates.io/crates/glommio), [crossbeam](https://crates.io/crates/crossbeam), parking_lot, dashmap, hashbrown, mimalloc, governor, moka |
| 2 — Serialization/Data | 19 | [serde](https://crates.io/crates/serde), capnp, bytes, uuid, url, http, smallvec, toml, [nom](https://crates.io/crates/nom), winnow |
| 3 — Transport | 18 | [quinn](https://crates.io/crates/quinn), [quiche](https://crates.io/crates/quiche), [hyper](https://crates.io/crates/hyper), h2, h3, [rustls](https://crates.io/crates/rustls), [boring](https://crates.io/crates/boring), [reqwest](https://crates.io/crates/reqwest), [tower](https://crates.io/crates/tower), pingora, rustls-platform-verifier |
| 4 — Observability | 13 | [tracing](https://crates.io/crates/tracing), tracing-journald, tracing-error, opentelemetry, prometheus-client, metrics, sentry |
| 5 — Error Handling | 4 | thiserror, anyhow, miette, eyre |
| 6 — CLI/Config | 6 | [clap](https://crates.io/crates/clap), chrono, time, humantime, dirs, dotenvy |
| 7 — Domain | 17 | hickory, notify, ipnet, [backon](https://crates.io/crates/backon), regex, [axum](https://crates.io/crates/axum), [ractor](https://crates.io/crates/ractor), flate2, cloudflare, foundations, self_update, tokio-console |
| 8 — Crypto/Security | 19 | ring, sha2, p256, pem, jsonwebtoken, crypto_box, x25519-dalek |
| 9 — Platform | 12 | [nix](https://crates.io/crates/nix), libc, systemd, windows-service, socket2, sysinfo |
| 10 — Testing | 14 | proptest, mockall, criterion, wiremock, tempfile, insta |
| 11 — Tooling/Guardrails | 4 | debtmap-cli, debtmap library, cargo-deny, cargo-audit |
| **Total** | **151** | |
| Rewrite candidates | 18 | SOCKS5, ICMP, stream relay, session mgr, flow/limiter, facades, etc. |
| Custom facades | 3 | logging, CLI, transport |
