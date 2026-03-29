# Proxy Data Taxonomy

Classification framework for all S1 atoms and catalogs along the
**proxy-data vs transport-control** analytical dimension.

Core insight: **Everything coming from or going to the tunnel, except the
tunnel's transport state machine, IS proxy data.**

This taxonomy adds a first-class analytical axis to the existing S1 catalog
set. It does not change atom membership counts, Jaccard clusters, or catalog
boundaries — it overlays a plane classification that makes the data-plane vs
control-plane boundary explicit throughout.

## Plane Definitions

### Proxy Data Plane

Everything dispatched through the tunnel on behalf of end-user traffic.
Upstream evidence from [cloudflared@2026.3.0](https://github.com/cloudflare/cloudflared/tree/2026.3.0):

- **HTTP requests** — `originProxy.ProxyHTTP()` via both HTTP/2 and QUIC streams
- **WebSocket streams** — `originProxy.ProxyHTTP(isWebsocket=true)` upgraded
  connections
- **TCP connections** — `originProxy.ProxyTCP()` for WARP routing
- **UDP sessions** — datagramsession (v2) and quic/v3 (v3) session data
- **ICMP packets** — datagram relay through QUIC
- **Session registration RPCs** — v2 `RegisterUdpSession` /
  `UnregisterUdpSession` serve the data plane by establishing proxy data
  channels (classified as proxy data per project decision)
- **Registration RPC frames** — `RegisterConnection`,
  `UnregisterConnection`, `waitForUnregister` — session-based proxy
  data that configures the connection identity
- **Configuration sync frames** — `SendLocalConfiguration`,
  `UpdateConfiguration` — session-based proxy data that mutates
  connection state
- **Graceful shutdown frames** — `GracefulShutdown` — session-based
  proxy data that terminates the proxy session

The proxy data plane encompasses: ingress rule matching, origin service
dispatch, stream piping, datagram muxing, flow control, and proxy-layer
error handling.

### Transport Control Plane

The tunnel's transport state machine — connection lifecycle management that
carries no end-user traffic. Upstream evidence:

- **RPC transport mechanics** — the control stream as a QUIC/HTTP2
  stream object; stream establishment, framing, and teardown managed
  by the transport library layer
- **Protocol selection** — `protocolFallback`, `selectNextProtocol`,
  edge protocol negotiation
- **Edge address rotation** — DNS/SRV-based address selection, reconnect
  signaling
- **Supervisor lifecycle** — HA slot tracking, backoff engine, connection
  state tracking, reconnect orchestration

### Out-of-Band

Components that operate outside the tunnel entirely — not on the tunnel wire
and not part of the tunnel's transport state machine:

- **Management service** — local HTTP server on `localhost`, not tunneled
- **Edge discovery** — DNS/SRV lookups (pre-tunnel establishment)
- **CLI interaction** — command parsing, subcommand dispatch, service
  installation
- **Diagnostics** — system/network/log collectors, diagnostic handlers
- **Metrics serving** — Prometheus endpoint on localhost
- **Upstream API** — cfapi REST calls for tunnel/route CRUD (API, not tunnel)

## Hub Atom Classification Index

All 47 hub atoms (membership ≥ 8) classified by plane. Classification is
based on each atom's primary behavioral role as documented in its S1 atom
file and verified against upstream source.

### Proxy Data Atoms

| Atom | Catalogs | Role |
| --- | ---: | --- |
| [carrier/carrier](atoms/carrier/carrier.md) | 10 | WebSocket carrier lifecycle |
| [quic/v3/session](atoms/quic/v3/session.md) | 12 | QUIC v3 session lifecycle |
| [quic/v3/muxer](atoms/quic/v3/muxer.md) | 10 | QUIC v3 stream muxer |
| [quic/v3/manager](atoms/quic/v3/manager.md) | 8 | QUIC v3 session manager |
| [quic/v3/metrics](atoms/quic/v3/metrics.md) | 8 | QUIC v3 metrics |
| [connection/quic_datagram_v2](atoms/connection/quic_datagram_v2.md) | 10 | QUIC datagram v2 transport |
| [connection/quic_datagram_v3](atoms/connection/quic_datagram_v3.md) | 10 | QUIC datagram v3 transport |
| [datagramsession/manager](atoms/datagramsession/manager.md) | 9 | Datagram session manager |
| [datagramsession/session](atoms/datagramsession/session.md) | 9 | Datagram session lifecycle |
| [datagramsession/metrics](atoms/datagramsession/metrics.md) | 8 | Datagram session metrics |
| [ingress/origin_service](atoms/ingress/origin_service.md) | 9 | Ingress origin service lifecycle |
| [ingress/icmp_linux](atoms/ingress/icmp_linux.md) | 8 | Linux ICMP proxy implementation |
| [ingress/icmp_darwin](atoms/ingress/icmp_darwin.md) | 8 | macOS ICMP proxy implementation |
| [stream/stream](atoms/stream/stream.md) | 9 | Bidirectional stream pipe |
| [tunnelrpc/quic/session_client](atoms/tunnelrpc/quic/session_client.md) | 10 | Session registration RPC client |
| [tunnelrpc/pogs/session_manager](atoms/tunnelrpc/pogs/session_manager.md) | 8 | Session manager RPC |
| [tunnelrpc/quic/cloudflared_client](atoms/tunnelrpc/quic/cloudflared_client.md) | 10 | Registration + config + shutdown RPC — session-based proxy data |
| [tunnelrpc/registration_client](atoms/tunnelrpc/registration_client.md) | 10 | Registration RPC client — session-based proxy data |
| [tunnelrpc/pogs/registration_server](atoms/tunnelrpc/pogs/registration_server.md) | 8 | Registration server RPC — session-based proxy data |
| [tunnelrpc/pogs/configuration_manager](atoms/tunnelrpc/pogs/configuration_manager.md) | 9 | Configuration manager RPC — session-based proxy data |
| [tunnelrpc/proto/tunnelrpc.capnp](atoms/tunnelrpc/proto/tunnelrpc.capnp.md) | 8 | Cap'n Proto RPC schema — defines all proxy data message formats |
| [tunnelrpc/metrics/metrics](atoms/tunnelrpc/metrics/metrics.md) | 8 | RPC instrumentation — measures proxy data operations |

### Transport Control Atoms

| Atom | Catalogs | Role |
| --- | ---: | --- |
| [connection/protocol](atoms/connection/protocol.md) | 13 | Protocol selection and fallback |
| [supervisor/tunnel](atoms/supervisor/tunnel.md) | 12 | Tunnel supervisor loop |
| [supervisor/supervisor](atoms/supervisor/supervisor.md) | 10 | Supervisor HA startup loop |
| [connection/tunnelsforha](atoms/connection/tunnelsforha.md) | 8 | HA tunnel slot management |
| [retry/backoffhandler](atoms/retry/backoffhandler.md) | 9 | Exponential backoff handler |
| [tlsconfig/tlsconfig](atoms/tlsconfig/tlsconfig.md) | 8 | TLS configuration and cipher selection |
| [connection/quic](atoms/connection/quic.md) | 9 | QUIC connection setup |

### Mixed Atoms (Serve Both Planes)

| Atom | Catalogs | Primary | Role |
| --- | ---: | --- | --- |
| [cmd/cloudflared/tunnel/configuration](atoms/cmd/cloudflared/tunnel/configuration.md) | 14 | mixed | Central dispatch — builds both control and proxy config |
| [connection/http2](atoms/connection/http2.md) | 10 | mixed | HTTP/2 connection — control stream + proxy streams share one conn |
| [connection/quic_connection](atoms/connection/quic_connection.md) | 10 | mixed | QUIC connection — control RPC + proxy streams share one conn |
| [orchestration/orchestrator](atoms/orchestration/orchestrator.md) | 10 | mixed | Config hot-reload (control) + proxy store (data) |
| [connection/observer](atoms/connection/observer.md) | 10 | mixed | Observes events from both planes |
| [connection/control](atoms/connection/control.md) | 14 | mixed | RPC control stream — carries proxy-data RPC frames; stream mechanics are transport-control |
| [connection/errors](atoms/connection/errors.md) | 9 | mixed | Connection errors — spans transport (EdgeQuicDial, StreamListener) and proxy data (registration, datagram) |

### Out-of-Band Atoms

| Atom | Catalogs | Role |
| --- | ---: | --- |
| [management/service](atoms/management/service.md) | 12 | Management service runtime (local HTTP) |
| [management/events](atoms/management/events.md) | 9 | Management event streaming |
| [management/session](atoms/management/session.md) | 9 | Management session lifecycle |
| [management/token](atoms/management/token.md) | 9 | Management API token handling |
| [cmd/cloudflared/tunnel/cmd](atoms/cmd/cloudflared/tunnel/cmd.md) | 13 | Primary tunnel CLI command |
| [cmd/cloudflared/main](atoms/cmd/cloudflared/main.md) | 8 | CLI entrypoint |
| [cmd/cloudflared/tunnel/subcommands](atoms/cmd/cloudflared/tunnel/subcommands.md) | 9 | Tunnel subcommand dispatch |
| [cmd/cloudflared/tunnel/quick_tunnel](atoms/cmd/cloudflared/tunnel/quick_tunnel.md) | 8 | Quick tunnel provisioning |
| [cmd/cloudflared/windows_service](atoms/cmd/cloudflared/windows_service.md) | 10 | Windows service lifecycle |
| [config/configuration](atoms/config/configuration.md) | 10 | Configuration file loading |
| [connection/header](atoms/connection/header.md) | 8 | Connection header management |

### Classification Summary

| Plane | Hub atoms | Combined catalog memberships |
| --- | ---: | ---: |
| Proxy data | 22 | 199 |
| Transport control | 7 | 69 |
| Mixed | 7 | 77 |
| Out-of-band | 11 | 105 |
| **Total** | **47** | **450** |

## Per-Catalog Impact Assessment

Each of the 30 catalogs classified by how much the proxy-data framing
changes its analytical lens. **High**: needs structural annotation or
reframing. **Medium**: needs section-level plane labels or scope
clarification. **Low**: needs only a brief taxonomy reference note.

### High Impact (11 catalogs — core data/control boundary)

| Catalog | Type | Primary Plane | Change |
| --- | --- | --- | --- |
| [tunnels](catalogs/domain/tunnels.md) | domain | mixed | Split domain map into control/data groupings |
| [proxying](catalogs/domain/proxying.md) | domain | proxy-data | Strengthen "IS proxy data" framing |
| [tunnels-transport](catalogs/domain/tunnels-transport.md) | domain | mixed | Add proxy-data vs control column to transport matrix |
| [sessions](catalogs/domain/sessions.md) | domain | proxy-data | Reframe as UDP proxy data lifecycle |
| [capnp-rpc](catalogs/domain/capnp-rpc.md) | domain | mixed | Reclassify registration/config/shutdown RPC frames as session-based proxy data; transport mechanics remain transport-control; split domain map to show both planes |
| [wire-protocol](catalogs/cross-cutting/wire-protocol/README.md) | cross-cutting | mixed | Add plane column to communication class matrix |
| [state-machines](catalogs/domain/state-machines.md) | domain | mixed | Tag each state machine by plane |
| [supervisor](catalogs/domain/supervisor.md) | domain | transport-control | Reframe as transport-control orchestration |
| [edge-interactions](catalogs/domain/edge-interactions.md) | domain | mixed | Add plane framing to discovery vs data flow |
| [upstream-api-contracts](catalogs/domain/upstream-api-contracts.md) | domain | out-of-band | Label API families by plane |
| [ingress](catalogs/domain/ingress.md) | domain | proxy-data | Label request flow as proxy data path |

### Medium Impact (10 catalogs — cross-cutting lenses)

| Catalog | Type | Primary Plane | Change |
| --- | --- | --- | --- |
| [concurrency](catalogs/cross-cutting/concurrency/README.md) | cross-cutting | mixed | Plane-label goroutine spawn sites |
| [error-propagation](catalogs/cross-cutting/error-propagation/README.md) | cross-cutting | mixed | Plane-annotate error classification trees |
| [init-teardown](catalogs/cross-cutting/init-teardown/README.md) | cross-cutting | mixed | Plane-label startup DAG dependencies |
| [shared-state](catalogs/domain/shared-state.md) | domain | mixed | Annotate shared state by plane ownership |
| [features](catalogs/cross-cutting/features/README.md) | cross-cutting | mixed | Plane-label stakeholder contracts |
| [porting-friction](catalogs/cross-cutting/porting-friction/README.md) | cross-cutting | mixed | Plane-annotate Go→Rust idiom translations |
| [config](catalogs/domain/config.md) | domain | mixed | Classify config values by plane |
| [metrics](catalogs/domain/metrics.md) | domain | mixed | Classify metrics by plane |
| [observabilities](catalogs/domain/observabilities.md) | domain | mixed | Classify traces by plane |
| [const-and-env](catalogs/domain/const-and-env.md) | domain | mixed | Note which constants serve which plane |

### Low Impact (9 catalogs — brief taxonomy reference)

| Catalog | Type | Primary Plane | Change |
| --- | --- | --- | --- |
| [cli](catalogs/domain/cli.md) | domain | out-of-band | Add taxonomy reference note |
| [crypto](catalogs/domain/crypto.md) | domain | transport-control | Note: TLS is transport infrastructure |
| [overwatch](catalogs/domain/overwatch.md) | domain | out-of-band | Add taxonomy reference note |
| [access-policies](catalogs/domain/access-policies.md) | domain | out-of-band | Add taxonomy reference note |
| [platforms](catalogs/domain/platforms.md) | domain | out-of-band | Add taxonomy reference note |
| [platform-substrates](catalogs/cross-cutting/platform-substrates.md) | cross-cutting | out-of-band | Add taxonomy reference note |
| [host-interactions](catalogs/domain/host-interactions.md) | domain | out-of-band | Add taxonomy reference note |
| [deployments](catalogs/domain/deployments/) | domain | out-of-band | Add taxonomy reference note |
| [tests](catalogs/cross-cutting/tests.md) | cross-cutting | mixed | Plane-annotate test contract tables |

### Impact Summary

| Impact | Count | Description |
| --- | ---: | --- |
| High | 11 | Structural annotation or section reframing |
| Medium | 10 | Section-level plane labels |
| Low | 9 | Brief taxonomy reference note only |
| **Total** | **30** | |

## Session Classification of Proxy Data

The proxy data plane divides into two behavioral subtypes observable
in the Go codebase — distinguished by whether the Go code manages
explicit session state or dispatches statelessly per request.

### Subtype Definitions

**Session-Based Proxy Data**
Proxy data that explicitly allocates, tracks, or tears down session
identifiers or session state over time. Requires a session registry,
lifecycle management, or identity tracking.

**Sessionless Proxy Data**
Proxy data that operates per request or per stream without an explicit
long-lived session registry in the behavioral contract. Stateless after
any initial handshake.

### Session Classification Table

All proxy data atoms classified by session subtype.

| Atom | Session Subtype | Evidence |
| --- | --- | --- |
| `quic/v3/session` | session-based | RequestID tracking, migration, idle timeout |
| `quic/v3/muxer` | session-based | Session dispatch registry, deconfliction |
| `quic/v3/manager` | session-based | Manager registration, session sweep |
| `connection/quic_datagram_v2` | session-based | Per-session goroutine loops, RPC registration |
| `connection/quic_datagram_v3` | session-based | Manager/muxer model, RequestID |
| `datagramsession/session` | session-based | UDP session lifecycle, idle timeout, 3 close paths |
| `datagramsession/manager` | session-based | Session registry, bidirectional relay |
| `tunnelrpc/quic/session_client` | session-based | RegisterUdpSession/UnregisterUdpSession lifecycle |
| `tunnelrpc/pogs/session_manager` | session-based | Session registration RPC marshaling |
| `ingress/icmp_linux` | session-based | Echo-ID tracking, 2 router goroutines, shared table |
| `ingress/icmp_darwin` | session-based | Echo-ID tracking, macOS-specific router state |
| `tunnelrpc/quic/cloudflared_client` | session-based | RegisterConnection identity, UpdateLocalConfiguration, GracefulShutdown |
| `tunnelrpc/registration_client` | session-based | Connection registration lifecycle, explicit teardown |
| `tunnelrpc/pogs/registration_server` | session-based | Registration server, explicit connection identity |
| `tunnelrpc/pogs/configuration_manager` | session-based | State mutation on existing session |
| `carrier/carrier` | sessionless | Per-connection WebSocket pipe, no session registry |
| `stream/stream` | sessionless | Bidirectional byte pipe, stateless after setup |
| `ingress/origin_service` | sessionless | Per-request origin dispatch, no session identity |
| `connection/http2` (proxy streams only) | sessionless | Per-request HTTP/2 streams, no session registry |
| `quic/v3/metrics` | session-based | Measures QUIC v3 session operations |
| `datagramsession/metrics` | session-based | Measures datagram session operations |
| `tunnelrpc/proto/tunnelrpc.capnp` | session-based | Defines all session-based RPC message formats |
| `tunnelrpc/metrics/metrics` | session-based | Instruments session-based RPC calls |

Note: Metrics and codec atoms (`quic/v3/metrics`, `datagramsession/metrics`,
`tunnelrpc/metrics/metrics`, `tunnelrpc/proto/tunnelrpc.capnp`) are
session-classified but are stateless instrumentation and type definitions
— they do not manage session state themselves.

Note: `connection/http2` and `connection/quic_connection` are mixed atoms.
Their control-stream aspect is transport-control. Their proxy-stream
aspect is sessionless proxy data.

### Go Dispatch Model

| Dispatch Pattern | Handles | Go Behavioral Character |
| --- | --- | --- |
| **Session-managed goroutines** | All session-based proxy data | Long-lived goroutines with session registries, explicit lifecycle teardown |
| **Per-request handlers** | All sessionless proxy data | Short-lived per-request/stream goroutines, no session registry |
| **Transport state machine** | QUIC mechanics, TLS, HTTP/2 framing | Connection-level goroutines managing transport lifecycle — not proxy data dispatch |

### Session-Managed Atom Set

These atoms use long-lived goroutines with session registries in Go:

- QUIC datagram v2 + v3: `connection/quic_datagram_v2`,
  `connection/quic_datagram_v3`, `quic/v3/session`, `quic/v3/muxer`,
  `quic/v3/manager`
- UDP session management: `datagramsession/session`,
  `datagramsession/manager`
- Session RPC: `tunnelrpc/quic/session_client`,
  `tunnelrpc/pogs/session_manager`
- ICMP routing: `ingress/icmp_linux`, `ingress/icmp_darwin`
- capnp-rpc control stream: `tunnelrpc/quic/cloudflared_client`,
  `tunnelrpc/registration_client`, `tunnelrpc/pogs/registration_server`,
  `tunnelrpc/pogs/configuration_manager`

### Per-Request Dispatch Atom Set

These atoms use per-request/per-stream goroutine handlers in Go:

- Stream relay: `stream/stream`, `carrier/carrier`
- Origin dispatch: `ingress/origin_service`
- HTTP/2 proxy streams (data-plane only): `connection/http2`
- WebSocket carrier streams: `carrier/websocket` (not a hub atom
  but implied by carrier/carrier classification)
- TCP proxy streams: `ingress/origin_connection` (implied by
  origin_service classification)
- SOCKS5: `socks/connection_handler` (not a hub atom, sessionless
  by S1 Session Orientation classification)

### Notes on Mixed Atoms

`connection/http2` and `connection/quic_connection` span both dispatch
models because they carry both the transport state machine and proxy
streams on the same connection object. The Go dispatch boundary is:

- Connection establishment + control stream → transport state machine goroutines
- Per-request proxy streams → per-request handler goroutines
- Datagram sessions → session-managed goroutines with registries

## Editorial Rubric

Rules for how the proxy-data framing gets applied across catalogs.

### What Changes

1. **Scope sections** — Add a one-line plane classification statement:
   "This catalog primarily covers the **proxy data plane** / **transport
   control plane** / **both planes** / **out-of-band infrastructure**."
   Link to this taxonomy: [proxy-data-taxonomy](proxy-data-taxonomy.md).

2. **Domain maps and architecture diagrams** — Where a domain map or
   Mermaid diagram mixes proxy-data and transport-control elements, add
   inline annotations (`[proxy-data]` / `[transport-control]`) or split
   into labeled subsections. Only restructure where the current layout
   actively obscures the plane boundary (primarily
   [tunnels](catalogs/domain/tunnels.md) domain map split).

3. **Tables and matrices** — Where classification tables list mixed-plane
   items (wire-protocol communication classes, state machine inventory,
   RPC schema), add a "Plane" column with values: `proxy-data`,
   `transport-control`, `out-of-band`, or `mixed`.

4. **Cross-references** — Use consistent terminology: "proxy data" (not
   "data plane" or "user traffic"), "transport control" (not "control
   plane" or "tunnel management"), "out-of-band" (not "external" or
   "auxiliary").

### What Does Not Change

- **Atom boundaries** — No atoms are added, removed, or split
- **Catalog boundaries** — No catalogs are merged or split
- **Membership counts** — Hub atom catalog memberships remain unchanged
- **Jaccard clusters** — The 8 natural clusters remain valid
- **Section structure** — Existing Scope / Architecture / Domain Map /
  Upstream-Verified / Notes skeleton is preserved
- **Atom files** — Individual atom documents in `atoms/` are not modified
  (catalog-level framing is sufficient)

### Terminology

| Term | Meaning | Do not use |
| --- | --- | --- |
| proxy data | End-user traffic through the tunnel | data plane, user traffic |
| transport control | Tunnel state machine lifecycle | control plane, tunnel management |
| out-of-band | Not on the tunnel wire | external, auxiliary |
| mixed | Atom or catalog serves both proxy data and transport control | hybrid, dual |

## Notes

- This taxonomy is an S1 revision triggered by S2 analytical needs,
  following the RR feedback loop: S2→S1 lightweight patch
  (see [README](../README.md))
- Session registration RPCs (v2: `RegisterUdpSession` /
  `UnregisterUdpSession`) are classified as **proxy data** — they establish
  proxy data channels even though they use the control stream transport
- The `connection/header` atom is classified as out-of-band because its
  role is HTTP header manipulation infrastructure, not directly
  proxy-data-bearing or transport-control-specific
- The classification of mixed atoms acknowledges that some implementation
  files genuinely serve both planes — this is a structural reality of the
  cloudflared codebase, not a taxonomy weakness
- capnp-rpc registration, configuration, and shutdown frames
  (`RegisterConnection`, `UpdateLocalConfiguration`, `GracefulShutdown`)
  are classified as **proxy data (session-based)** — they travel through
  the tunnel as bytes and carry session identity, state mutations, and
  explicit teardown semantics. The RPC *transport mechanics* (the QUIC
  stream object carrying them) remain transport-control. The distinction
  is: transport-control = how bytes move; proxy data = what the bytes are.
- The Cap'n Proto schema (`tunnelrpc/proto/tunnelrpc.capnp`) is classified
  as **proxy data** — it defines message formats for all RPC operations
  that flow through the tunnel wire (RegisterConnection, RegisterUdpSession,
  UpdateConfiguration, GracefulShutdown, etc.). It is a proxy data codec.
- `tunnelrpc/metrics/metrics` is classified as **proxy data** for
  consistency with `quic/v3/metrics` and `datagramsession/metrics` —
  all three measure proxy data operations. The RPC operations being
  instrumented are proxy data (they flow through the tunnel).
- `connection/errors` is classified as **mixed** — its error types span
  both transport events (`EdgeQuicDialError`, `StreamListenerError`) and
  proxy data events (`DupConnRegisterTunnelError`,
  `ServerRegisterTunnelError`, `DatagramManagerError`).
