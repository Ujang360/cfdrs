# Phase 3.6 — Ingress Proxy

| | |
| --- | --- |
| Stage | 3.6 |
| Scope | Request proxying, ingress rules, origin services, SOCKS5, carrier |
| Entry condition | Stage 3.3 exit gate passed AND stage 3.5 exit gate passed |
| Exit condition | All 37 Must-tier contracts green |

---

## Crates

| Order | Crate | Group | Must | Should | Fuzz | Total |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | `tunnel-ingress-proxy` | tunnel/ | 37 | 26 | 1 | 64 |

### Notes

Single-crate stage but the second-highest contract density (64
contracts, 37 Must-tier). This is the fattest behavioral crate in
the architecture — it houses ingress rule matching, origin service
taxonomy, HTTP/WS/TCP/SOCKS5 proxy dispatch, WebSocket carrier
lifecycle, and access JWT validation.

---

## Parity Gate

**Must-tier contracts (37):**

- `PROXY.single_origin_*` — HTTP/WS/SSE proxying
- `PROXY.parse_ingress_table_driven` — ingress rule parsing
- `PROXY.rule_matches_*` — hostname matching
- `PROXY.conn_*` — cross-protocol proxying
- `PROXY.socks5_*` — SOCKS5 proxying
- `PROXY.stream_tcp_bidirectional` — TCP relay
- `PROXY.raw_tcp_service_connect_fail` — TCP connect error path
- `PROXY.tcp_over_ws_service_connect` — TCP-over-WebSocket
- `PROXY.error_propagation_502` — error → HTTP status mapping
- Additional origin service and proxy contracts

**Fuzz targets (1):** Ingress rule parsing robustness.

All 37 Must contracts must be `green` before stage 3.8 begins.

---

## Key Deliverables

### `ingress` module

- Ingress rule matching (hostname glob, path regex, catch-all,
  punycode, port stripping, internal rules negative index)
- Origin service taxonomy (http, https, tcp, unix, socks-proxy,
  bastion, hello-world, http_status, DNS)
- 4-layer config merge precedence (pointer-nil gating quirk)
- JWT middleware (`SkipClientIDCheck` quirk)
- IP access policy enforcement
- Path-in-origin-URL rejection quirk

### `proxy` module

- Fat enum dispatch (ARCH-1, ARCH-4 — no `Box<dyn>` on hot path)
- HTTP proxy (SSE/gRPC flushing, header canonicalization)
- WebSocket proxy
- TCP stream relay
- SOCKS5 inbound server (NoAuth + UserPassAuth, CONNECT only)
- Hello-world server
- Error → HTTP status mapping (502, 504, 404)

### `carrier` module

- WebSocket carrier lifecycle for access forwarding
- stdio stream wrapper
- Access control-plane HTTP request builder
- Bastion header routing
- WebSocket upgrade orchestration

### `validation` module

- Access JWT validation, OIDC token verification
- URL/hostname sanitization (RFC labels, IDN, scheme/port, IP rules)

---

## Invariants Active

| ID | Invariant | Enforcement |
| --- | --- | --- |
| ARCH-1 | Proxy data fat enum — no `Box<dyn>` on hot path | Compile-time |
| ARCH-4 | Fat enum dispatch — exhaustive match | Compile-time |
| ARCH-5 | Bump arena discipline for session temporaries | Allocation policy |
| PROXY-1 | Ingress table parsing parity | `PROXY.parse_ingress_table_driven` |
| PROXY-2 | Rule matching with hostname/path dispatch | `PROXY.rule_matches_12_cases` |
| ERR-2 | Error propagation → 502 | `PROXY.error_propagation_502` |

---

## Risks

| Risk | Description | Mitigation |
| --- | --- | --- |
| R1.1 | Trait interface gaps in proxy dispatch | Interfaces locked in 3.0 |
| R2.4 | Fire-and-forget goroutines in WebSocket carrier | Bounded periodic tasks with timeout |
| R1.5 | `defer`-based cleanup in proxy paths | Explicit drop semantics |

---

## Session Guidance

- **Session count:** 1–2 sessions (single crate, high density)
- **Cognitive load:** High — fattest behavioral crate; four modules
  with distinct concerns; 37 Must-tier contracts to satisfy
- **Critical path:** On the critical path (3.5 → **3.6** → 3.8)
- **Buffer pool decision:** S3 must resolve origin-facing buffer
  strategy (reuse pool vs `BufFactory` extension vs per-request
  allocation) — see [architecture](../architecture.md)
  `tunnel-ingress-proxy` S3 note
- **Cargo check:** `cargo check --workspace` after completion
