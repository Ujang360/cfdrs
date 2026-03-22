# Features — API, Monitoring, Origin, and DNS Stakeholders

> Part of the [Features Behavior Catalog](README.md).

## Stakeholder 3 — Cloudflare Dashboard & API

The dashboard and API gateway interact with cloudflared through two channels: the cfapi REST client (used by the CLI for control-plane operations) and the management HTTP/WebSocket service (used by the dashboard for runtime inspection).

### cfapi REST Client

All tunnel-management operations go through the cfapi client which wraps Cloudflare's v4 API.

| Resource | Operations | Atoms |
| --- | --- | --- |
| Tunnel | Create, Get, List, Delete, Token, ManagementToken, CleanupConnections | [cfapi/tunnel](../../../atoms/cfapi/tunnel.md), [cfapi/tunnel_filter](../../../atoms/cfapi/tunnel_filter.md) |
| IP Route | Add, Delete, List, Get | [cfapi/ip_route](../../../atoms/cfapi/ip_route.md), [cfapi/ip_route_filter](../../../atoms/cfapi/ip_route_filter.md) |
| Virtual Network | Create, Delete, List, Update | [cfapi/virtual_network](../../../atoms/cfapi/virtual_network.md), [cfapi/virtual_network_filter](../../../atoms/cfapi/virtual_network_filter.md) |
| DNS Hostname | Route, List | [cfapi/hostname](../../../atoms/cfapi/hostname.md) |
| Base HTTP | Auth headers, pagination, error handling | [cfapi/base_client](../../../atoms/cfapi/base_client.md), [cfapi/client](../../../atoms/cfapi/client.md) |

Deeper API contract documentation: [upstream-api-contracts](../upstream-api-contracts.md).

### Management Service (Dashboard → Connector)

The management service is an HTTP server embedded in each connector, accessible through the edge management tunnel on a configurable hostname (default: `management.argotunnel.com`). Requests are authenticated via scoped JWT tokens obtained from the cfapi management-token endpoint.

| Endpoint | Method | Purpose | Authentication |
| --- | --- | --- | --- |
| `/ping` | GET, HEAD | Liveness check. | Access token query param. |
| `/logs` | GET (WebSocket) | Live log streaming with filters. | Access token → JWT claims → actor. |
| `/host_details` | GET | Returns connector ID, private IP, hostname/label. | Access token query param. |
| `/metrics` | GET | Prometheus metrics (when `management-diagnostics` enabled). | Access token query param. |
| `/debug/pprof/*` | GET | Go pprof profiling endpoints. | Access token query param. |

Constraints:

- Maximum 1 concurrent streaming session; same actor can preempt their own session.
- CORS restricted to `*.cloudflare.com` origins.
- Idle timeout of 5 minutes when not actively streaming.
- WebSocket heartbeat ping every 15 seconds.

Primary evidence: [management/service](../../../atoms/management/service.md), [management/session](../../../atoms/management/session.md), [management/events](../../../atoms/management/events.md), [management/logger](../../../atoms/management/logger.md).

### Remote Configuration Push

When `TunnelIsRemotelyManaged` is `true` in the registration response, the edge can push configuration updates to the connector via the orchestrator.

| Contract | Detail | Atoms |
| --- | --- | --- |
| Config update channel | `Orchestrator.UpdateConfig(version, configBytes)` | [orchestration/orchestrator](../../../atoms/orchestration/orchestrator.md), [orchestration/config](../../../atoms/orchestration/config.md) |
| Local config export | Serialized JSON sent to edge via `SendLocalConfiguration` | [tunnelrpc/pogs/configuration_manager](../../../atoms/tunnelrpc/pogs/configuration_manager.md) |
| Config change watch | File watcher for local YAML changes | [watcher/file](../../../atoms/watcher/file.md) |

## Stakeholder 5 — Monitoring Systems

External monitoring systems interact with cloudflared through Prometheus-compatible metrics endpoints, a readiness/health server, and diagnostic HTTP handlers.

### Prometheus Metrics Endpoint

The metrics listener exposes a Prometheus `/metrics` endpoint on a configurable address (default: `localhost:` with known-port fallback).

| Aspect | Detail | Atoms |
| --- | --- | --- |
| Listener setup | `CreateMetricsListener` with known-address fallback | [metrics/metrics](../../../atoms/metrics/metrics.md), [metrics/config](../../../atoms/metrics/config.md) |
| Connection metrics | Tunnel registration counts, protocol distribution, server locations | [connection/metrics](../../../atoms/connection/metrics.md) |
| Supervisor metrics | Tunnel active/reconnecting counts | [supervisor/metrics](../../../atoms/supervisor/metrics.md) |
| Proxy metrics | Request counts, latency histograms, error rates | [proxy/metrics](../../../atoms/proxy/metrics.md) |
| Orchestration metrics | Config update counts and versions | [orchestration/metrics](../../../atoms/orchestration/metrics.md) |
| RPC metrics | Cap'n Proto operation counts, latencies, failures | [tunnelrpc/metrics/metrics](../../../atoms/tunnelrpc/metrics/metrics.md) |
| Flow metrics | Rate-limiter accept/reject counters | [flow/metrics](../../../atoms/flow/metrics.md) |
| Datagram session metrics | Session open/close, packet counts | [datagramsession/metrics](../../../atoms/datagramsession/metrics.md) |
| QUIC v3 metrics | Datagram v3 session and packet metrics | [quic/v3/metrics](../../../atoms/quic/v3/metrics.md) |

### Readiness & Health

The readiness server provides health-check endpoints that report whether at least one tunnel connection is active.

| Endpoint | Behavior | Atom |
| --- | --- | --- |
| `/ready` | Returns 200 when connected, 503 when not. Uses `ConnTracker` state. | [metrics/readiness](../../../atoms/metrics/readiness.md) |
| ConnTracker | Observes connection events and maintains active-connection state. | [tunnelstate/conntracker](../../../atoms/tunnelstate/conntracker.md) |

### Diagnostic HTTP Handlers

The diagnostic subsystem exposes structured data about system state, tunnel state, and CLI configuration.

| Handler | Path | Response | Atom |
| --- | --- | --- | --- |
| System information | `/diag/system` | OS version, memory, disk info (platform-specific collector). | [diagnostic/handlers](../../../atoms/diagnostic/handlers.md), [diagnostic/diagnostic](../../../atoms/diagnostic/diagnostic.md) |
| Tunnel state | `/diag/tunnel` | Tunnel ID, connector ID, active connections, ICMP sources. | [diagnostic/handlers](../../../atoms/diagnostic/handlers.md) |
| Configuration | `/diag/configuration` | Non-secret CLI flags as JSON. | [diagnostic/handlers](../../../atoms/diagnostic/handlers.md) |
| Diagnostic client | Network | Remote diagnostic queries from CLI `tunnel diagnose`. | [diagnostic/client](../../../atoms/diagnostic/client.md) |

### Connection-Aware Logging

The supervisor uses a connection-aware logger that enriches log messages with tunnel connection state, enabling operators and monitoring systems to correlate logs with HA connection indices.

Primary evidence: [supervisor/conn_aware_logger](../../../atoms/supervisor/conn_aware_logger.md).

## Stakeholder 6 — Origin Services

Origin services are the local applications that cloudflared proxies traffic to. The interaction boundary is defined by ingress rules and the proxy layer.

### Ingress Rule Engine

Incoming requests from the edge are matched against an ordered list of ingress rules. Each rule specifies a hostname pattern, path regex, and an origin service target.

| Concept | Detail | Atoms |
| --- | --- | --- |
| Rule set | Ordered list with hostname glob + path regex → service | [ingress/ingress](../../../atoms/ingress/ingress.md), [ingress/rule](../../../atoms/ingress/rule.md) |
| Origin service types | HTTP(S), WebSocket, TCP, Unix socket, hello-world, bastion | [ingress/origin_service](../../../atoms/ingress/origin_service.md) |
| Origin config | `originRequest` block with timeouts, TLS, headers, proxy settings | [ingress/config](../../../atoms/ingress/config.md) |

Deeper ingress documentation: [ingress](../ingress.md).

### Proxy Layer

The proxy layer handles request/response forwarding between the tunnel connection and origin services.

| Protocol | Behavior | Atoms |
| --- | --- | --- |
| HTTP proxy | Standard request/response with header canonicalization, flushing for SSE/gRPC | [proxy/proxy](../../../atoms/proxy/proxy.md) |
| WebSocket proxy | Protocol upgrade → bidirectional byte stream | [websocket/websocket](../../../atoms/websocket/websocket.md), [websocket/connection](../../../atoms/websocket/connection.md) |
| TCP proxy | Raw TCP byte stream via `ReadWriteAcker` | [connection/connection](../../../atoms/connection/connection.md), [stream/stream](../../../atoms/stream/stream.md) |
| SOCKS5 proxy | SOCKSv5 with auth negotiation → TCP dial to target | [socks/connection_handler](../../../atoms/socks/connection_handler.md), [socks/request_handler](../../../atoms/socks/request_handler.md) |
| ICMP proxy | ICMP echo request/reply tunneling (platform-specific) | [ingress/origin_icmp_proxy](../../../atoms/ingress/origin_icmp_proxy.md), [ingress/packet_router](../../../atoms/ingress/packet_router.md) |

### Hello-World Server

The built-in hello-world server provides a test origin that serves a static HTML page, useful for validating tunnel connectivity without an external origin.

Primary evidence: [hello/hello](../../../atoms/hello/hello.md).

### Origin Connection Management

| Aspect | Detail | Atoms |
| --- | --- | --- |
| Origin dialing | HTTP transport with configurable timeouts, keep-alive, happy-eyeballs | [ingress/origin_dialer](../../../atoms/ingress/origin_dialer.md) |
| Connection pooling | Idle connection limits and timeout management | [ingress/origin_connection](../../../atoms/ingress/origin_connection.md) |
| Origin TLS | Custom CA pool, server-name override, TLS verification toggle | [ingress/origin_proxy](../../../atoms/ingress/origin_proxy.md) |
| IP access rules | Source-IP allowlist/blocklist for origin access control | [ipaccess/access](../../../atoms/ipaccess/access.md) |

## Stakeholder 7 — DNS Infrastructure

DNS interactions drive both feature-flag rollout and edge-address discovery. cloudflared makes DNS queries as part of its startup and periodic refresh cycles.

### Feature-Flag DNS Contract

Feature rollout is controlled via a DNS TXT record at `cfd-features.argotunnel.com`. The record payload is a JSON object parsed into `featuresRecord`.

| Field | Type | Meaning |
| --- | --- | --- |
| `dv3_2` | `uint32` | Percentage of accounts (0–100) that should use datagram v3. |

The feature selector hashes the account tag with FNV-32a and compares `hash % 100 < percentage` to determine eligibility. Deprecated fields (`pq`, `dv3`, `dv3_1`) are ignored.

| Aspect | Detail | Atoms |
| --- | --- | --- |
| DNS resolver | `net.DefaultResolver.LookupTXT` with 10-second timeout | [features/selector](../../../atoms/features/selector.md) |
| Refresh cadence | Every `defaultLookupFreq` (1 hour) via background goroutine | [features/selector](../../../atoms/features/selector.md) |
| Feature dedup | `dedupAndRemoveFeatures` strips deprecated and duplicate entries | [features/features](../../../atoms/features/features.md) |

### Edge Address Discovery

Edge IP addresses are resolved via DNS SRV records to discover available Cloudflare edge servers for tunnel connections.

| Aspect | Detail | Atoms |
| --- | --- | --- |
| SRV resolution | Queries for edge server addresses by region and IP version | [edgediscovery/edgediscovery](../../../atoms/edgediscovery/edgediscovery.md), [edgediscovery/allregions/discovery](../../../atoms/edgediscovery/allregions/discovery.md) |
| Region management | Address pools organized by region with used-by tracking | [edgediscovery/allregions/region](../../../atoms/edgediscovery/allregions/region.md), [edgediscovery/allregions/regions](../../../atoms/edgediscovery/allregions/regions.md) |
| Address selection | Connectivity-aware address rotation with error tracking | [edgediscovery/allregions/address](../../../atoms/edgediscovery/allregions/address.md), [edgediscovery/allregions/usedby](../../../atoms/edgediscovery/allregions/usedby.md) |
| Edge dialing | Dial with protocol awareness and fallback | [edgediscovery/dial](../../../atoms/edgediscovery/dial.md) |
