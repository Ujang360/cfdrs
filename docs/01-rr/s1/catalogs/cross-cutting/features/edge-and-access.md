# Features — Edge and Access Stakeholders

> Part of the [Features Behavior Catalog](README.md).

## Stakeholder 2 — Cloudflare Edge

The Cloudflare edge is the primary counterpart for all tunnel connections. It handles registration, serves as the RPC server, relays proxied traffic, and pushes remote configuration updates. cloudflared acts as the client in this relationship.

### Registration Contract

Each HA connection registers itself with the edge via Cap'n Proto RPC. The registration carries authentication credentials, a tunnel ID, a connection index, and a `ConnectionOptions` payload containing client metadata.

| Field | Type | Semantic |
| --- | --- | --- |
| `TunnelAuth` | `{AccountTag, TunnelSecret}` | Authenticates the connector to the edge per-tunnel. |
| `TunnelID` | `uuid.UUID` | Identifies the named tunnel being connected. |
| `ConnIndex` | `uint8` | HA connection index (0 to `HAConnections-1`). |
| `ConnectionOptions.Client` | `ClientInfo{ClientID, Features, Version, Arch}` | Advertises connector capabilities, version, and CPU architecture. |
| `ConnectionOptions.OriginLocalIP` | `net.IP` | Reports the connector's local IP for operational metadata. |
| `ConnectionOptions.NumPreviousAttempts` | `uint8` | Informs edge of retry count for backoff coordination. |

Response: `ConnectionDetails{UUID, Location, TunnelIsRemotelyManaged}` on success; `ConnectionError{Cause, ShouldRetry, RetryAfter}` on failure.

Primary evidence: [tunnelrpc/pogs/registration_server](../../../atoms/tunnelrpc/pogs/registration_server.md), [tunnelrpc/registration_client](../../../atoms/tunnelrpc/registration_client.md).

### Feature Negotiation Contract

The `Features` list in `ClientInfo` advertises what capabilities this connector supports. The edge uses this to enable matching behavior on its side.

| Feature string | Meaning | Selection logic |
| --- | --- | --- |
| `serialized_headers` | Connector supports serialized header format. | Always on (default feature). |
| `quick_reconnects` | Connector supports quick reconnect flow. | Always on (default feature). |
| `allow_remote_config` | Connector accepts remotely-managed configuration. | Always on (default feature). |
| `support_datagram_v2` | Connector supports UDP/ICMP datagram v2 protocol. | Selected when v3 is not active. |
| `support_datagram_v3_2` | Connector supports datagram v3 (current revision). | Selected by DNS percentage or `--features support_datagram_v3_2` CLI flag. |
| `postquantum` | Connector uses post-quantum key exchange. | Selected by `--post-quantum` flag (strict mode). |
| `management_logs` | Connector supports management log streaming. | Always on (default feature). |
| `support_quic_eof` | Connector supports QUIC EOF signaling. | Always on (default feature). |

Primary evidence: [features/features](../../../atoms/features/features.md), [features/selector](../../../atoms/features/selector.md).

### Protocol Selection Contract

The connector negotiates the transport protocol with the edge. Protocol selection follows a percentage-based rollout mechanism with fallback.

| Protocol | Preference | Fallback behavior |
| --- | --- | --- |
| QUIC | Default first choice | Falls back to HTTP2 after `MaxEdgeAddrRetries` failures. |
| HTTP2 | Secondary | Used when QUIC is unavailable or after fallback. |

The protocol selector maintains state per-connection and supports mid-session fallback via `protocolFallback` tracked per HA index.

Primary evidence: [connection/protocol](../../../atoms/connection/protocol.md), [edgediscovery/protocol](../../../atoms/edgediscovery/protocol.md).

### Control Stream Operations

Once registered, the control stream supports:

| Operation | Direction | Semantic |
| --- | --- | --- |
| `RegisterConnection` | connector → edge | Initial registration with auth and features. |
| `UnregisterConnection` | connector → edge | Graceful shutdown with configurable grace period. |
| `UpdateLocalConfiguration` | connector → edge | Sends current local configuration as JSON for edge awareness. |
| `SendLocalConfiguration` | connector → edge | Alias for UpdateLocalConfiguration in the RPC client. |

Primary evidence: [connection/control](../../../atoms/connection/control.md), [tunnelrpc/registration_client](../../../atoms/tunnelrpc/registration_client.md), [tunnelrpc/registration_server](../../../atoms/tunnelrpc/registration_server.md).

### Connection Events

The `Observer` pattern broadcasts connection lifecycle events to registered sinks (e.g., `ConnTracker` for health checks, `TunnelState` for diagnostics).

| Event | Trigger |
| --- | --- |
| `RegisteringTunnel` | Connection dial begins. |
| `Connected` | Registration succeeded; includes protocol, location, edge address. |
| `Reconnecting` | Connection lost; reconnect loop entered. |
| `Unregistering` | Graceful shutdown initiated. |
| `Disconnected` | Connection fully closed. |
| `SetURL` | Quick-tunnel URL obtained. |

Primary evidence: [connection/event](../../../atoms/connection/event.md), [connection/observer](../../../atoms/connection/observer.md).

## Stakeholder 4 — Cloudflare Access

Cloudflare Access is the authentication and authorization layer used to protect both origin applications behind the tunnel and the management service itself.

### Access Login Flow

The `cloudflared access login` command initiates a browser-based OAuth flow to obtain an origin certificate (`cert.pem`) that proves account membership.

| Step | Behavior | Atoms |
| --- | --- | --- |
| Browser launch | Platform-specific browser launcher (darwin, unix, windows, other fallback). | [token/token](../../../atoms/token/token.md), [cmd/cloudflared/tunnel/login](../../../atoms/cmd/cloudflared/tunnel/login.md) |
| Token transfer | Local HTTP callback server receives token from browser redirect. | [token/transfer](../../../atoms/token/transfer.md), [token/path](../../../atoms/token/path.md) |
| Token encryption | Optional lock-file encryption for stored tokens. | [token/encrypt](../../../atoms/token/encrypt.md) |
| Credential storage | Origin cert written to default or specified path. | [credentials/origin_cert](../../../atoms/credentials/origin_cert.md) |

### Access Application Tunneling

The `access` command family tunnels application-layer protocols through Access-protected endpoints.

| Protocol | Command | Transport | Atom |
| --- | --- | --- | --- |
| HTTP(S) | `access curl` | Carrier with token injection | [cmd/cloudflared/access/cmd](../../../atoms/cmd/cloudflared/access/cmd.md) |
| SSH | `access ssh` | WebSocket carrier | [carrier/carrier](../../../atoms/carrier/carrier.md), [carrier/websocket](../../../atoms/carrier/websocket.md) |
| RDP | `access rdp` | WebSocket carrier | [cmd/cloudflared/access/carrier](../../../atoms/cmd/cloudflared/access/carrier.md) |
| SMB, TCP | `access smb`, `access tcp` | WebSocket carrier | [carrier/carrier](../../../atoms/carrier/carrier.md) |

### JWT Validation Middleware

When Access policies protect an origin behind the tunnel, cloudflared can optionally validate Access JWTs on incoming requests before proxying.

| Contract | Detail | Atoms |
| --- | --- | --- |
| Middleware chain | Configurable per-ingress-rule middleware pipeline. | [ingress/middleware/middleware](../../../atoms/ingress/middleware/middleware.md) |
| JWT validation | Validates `Cf-Access-Jwt-Assertion` header against Access certs endpoint. | [ingress/middleware/jwtvalidator](../../../atoms/ingress/middleware/jwtvalidator.md) |
| Management token validation | Validates management-service access tokens from query params. | [management/token](../../../atoms/management/token.md), [management/middleware](../../../atoms/management/middleware.md) |

### Access Validation

Input validation for hostnames and URLs used in access commands.

Primary evidence: [cmd/cloudflared/access/validation](../../../atoms/cmd/cloudflared/access/validation.md), [validation/validation](../../../atoms/validation/validation.md).

## Stakeholder 9 — Peer Connections (HA)

cloudflared maintains multiple simultaneous connections (default 4) to the Cloudflare edge for high availability. These connections coordinate through shared state but do not directly communicate with each other — coordination is mediated by the supervisor and shared data structures.

### HA Connection Coordination

| Aspect | Detail | Atoms |
| --- | --- | --- |
| Connection index tracking | Each connection has a unique index (0 to `HAConnections-1`). | [connection/tunnelsforha](../../../atoms/connection/tunnelsforha.md), [supervisor/tunnelsforha](../../../atoms/supervisor/tunnelsforha.md) |
| Supervisor loop | Manages concurrent connection goroutines, backoff, and restart. | [supervisor/supervisor](../../../atoms/supervisor/supervisor.md), [supervisor/tunnel](../../../atoms/supervisor/tunnel.md) |
| Protocol fallback state | Per-connection protocol choice with independent fallback tracking. | [supervisor/pqtunnels](../../../atoms/supervisor/pqtunnels.md) |
| Connected fuse | `ConnectedFuse` interface signals first-connection-established. | [supervisor/fuse](../../../atoms/supervisor/fuse.md) |
| Backoff coordination | Per-connection exponential backoff with max-retry limits. | [retry/backoffhandler](../../../atoms/retry/backoffhandler.md) |
| Reconnect signal | External or stdin-triggered reconnect of a random connection. | [supervisor/external_control](../../../atoms/supervisor/external_control.md), [cmd/cloudflared/tunnel/signal](../../../atoms/cmd/cloudflared/tunnel/signal.md) |

### HA Startup Sequence

The supervisor starts the first connection, waits for successful registration, then starts the remaining connections in sequence with a 1-second registration interval. The first connection's negotiated protocol is propagated to subsequent connections.

### Connection State Tracking

| Aspect | Detail | Atoms |
| --- | --- | --- |
| ConnTracker | Tracks active/inactive state per connection index. | [tunnelstate/conntracker](../../../atoms/tunnelstate/conntracker.md) |
| Connection-aware logger | Enriches log output with connection state summary. | [supervisor/conn_aware_logger](../../../atoms/supervisor/conn_aware_logger.md) |
| Observer sinks | Multiple sinks receive connection events (tracker, UI, metrics). | [connection/observer](../../../atoms/connection/observer.md) |
