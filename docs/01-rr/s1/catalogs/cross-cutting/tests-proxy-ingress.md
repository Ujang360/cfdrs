# Tests — Proxy & Ingress Contracts

- Parent: [tests](tests.md)
- Baseline reference: [cloudflare/cloudflared/tree/2026.3.0](https://github.com/cloudflare/cloudflared/tree/2026.3.0)

## Scope

This sub-catalog documents behavioral contracts encoded in proxy routing, ingress rule parsing, origin services, ICMP proxying, SOCKS5, carrier/WebSocket, orchestration, and supervisor tests.

Packages covered: `proxy`, `ingress`, `orchestration`, `supervisor`, `socks`, `carrier`.

---

## proxy/proxy\_test.go

Source: [proxy/proxy\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/proxy/proxy_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestProxySingleOrigin` | HTTP, WebSocket, and SSE sub-tests verify single-origin proxy behavior |
| `TestProxySSEAllData` | Regression: SSE stream delivers all data even when origin closes without final delimiter |
| `TestProxyMultipleOrigins` | Hostname-based routing dispatches to `api`/`hello-world`/`health`/wildcard origins |
| `TestProxyError` | Transport error at origin propagates as 502 with error message |
| `TestConnections` | 8 connection type permutations validate end-to-end proxy behavior |

### Connection Type Permutation Matrix

The `TestConnections` test is the most comprehensive proxy test. It validates all supported connection type combinations:

| Sub-Test | Eyeball → cfd | cfd → Origin | Key Assertion |
| --- | --- | --- | --- |
| `ws-ws` | WebSocket | WebSocket | Bidirectional echo |
| `tcp-tcp` | TCP | TCP | Bidirectional echo |
| `tcp-ws` | TCP | WebSocket | Cross-protocol proxying |
| `ws-tcp` | WebSocket | TCP | Cross-protocol proxying |
| `http-(ws)tcp` | HTTP | WebSocket-over-TCP | HTTP request proxied through WS-TCP bridge |
| `ws-ws different origin` | WebSocket | WebSocket (different URL) | Cross-origin WebSocket |
| `tcp closed origin` | TCP | TCP (closed) | Graceful error propagation |
| `tcp rate limited` | TCP | TCP (rate limited) | `ErrTooManyActiveFlows` metadata in response |

### Mock Types

| Type | Purpose |
| --- | --- |
| `mockHTTPRespWriter` | Captures HTTP response for assertion |
| `mockWSRespWriter` | Captures WebSocket upgrade response |
| `mockSSERespWriter` | Captures SSE event stream |
| `mockTCPRespWriter` | Captures TCP connection response |

### Key Behavioral Details

**gomock flow limiter**: The rate-limited test uses `mocks.NewMockLimiter` to simulate `ErrTooManyActiveFlows`, validating that the proxy returns a `ConnectResponse` with rate-limited metadata.

**SSE regression**: `TestProxySSEAllData` validates that when an origin closes the SSE stream without a trailing `\n\n` delimiter, all buffered data is still delivered to the client. This is an important edge case for the Rust port.

### Atom Links

- [proxy/proxy](../../atoms/proxy/proxy.md)

---

## proxy/proxy\_posix\_test.go

Source: [proxy/proxy\_posix\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/proxy/proxy_posix_test.go)

Build constraint: `//go:build !windows`

Contains POSIX-specific proxy test variants. Shares same test infrastructure as `proxy_test.go`.

### Atom Links

- [proxy/proxy](../../atoms/proxy/proxy.md)

---

## ingress/ingress\_test.go

Source: [ingress/ingress\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/ingress_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestParseUnixSocket` | Unix socket service parsed with `http` scheme |
| `TestParseUnixSocketTLS` | Unix+TLS socket service parsed with `https` scheme |
| `TestParseIngressNilConfig` | Nil config returns error |
| `TestParseIngress` | Table-driven: validates parsing of multiple rules, hostname patterns, unicode domains, wildcards, TCP/SSH/RDP/SMB/SOCKS/bastion services, http\_status, hello\_world, path regex, and error cases |
| `TestSingleOriginSetsConfig` | CLI flags propagate to ingress rule config fields |
| `TestSingleOriginServices` | CLI params map to correct OriginService types (hello\_world, bastion, http, tcp, unix) |
| `TestSingleOriginServices_URL` | HTTP/HTTPS URLs → httpService; SSH/RDP/SMB/TCP URLs → tcpOverWSService |
| `TestFindMatchingRule` | Host+path matching selects correct rule index |
| `TestIsHTTPService` | URL scheme classification: http/https/ws/wss → true; tcp → false |
| `TestParseAccessConfig` | Access config requires teamName when audTags present |
| `BenchmarkFindMatch` | Performance benchmark for rule matching |

### Key Behavioral Details

**Ingress rule parsing** is one of the most table-driven tests in the codebase. It encodes dozens of valid and invalid rule configurations via YAML, testing hostname patterns (exact, wildcard, unicode/punycode), service types (http, https, tcp, ssh, rdp, smb, socks-proxy, bastion, hello\_world, http\_status), path regex matching, and error detection for invalid URLs, missing catch-all rules, and duplicate rules.

**Service type mapping**: The test validates that CLI flags correctly map to internal service types:

| URL/Flag | Internal Service Type |
| --- | --- |
| `http://localhost` | `httpService` |
| `https://localhost` | `httpService` (with TLS) |
| `ssh://localhost` | `tcpOverWSService` (port 22) |
| `rdp://localhost` | `tcpOverWSService` (port 3389) |
| `smb://localhost` | `tcpOverWSService` (port 445) |
| `tcp://localhost` | `tcpOverWSService` |
| `bastion` | `bastionService` |
| `hello_world` | `helloWorld` |

### Atom Links

- [ingress/ingress](../../atoms/ingress/ingress.md)

---

## ingress/rule\_test.go

Source: [ingress/rule\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/rule_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `Test_rule_matches` | 12 cases: hostname matching, unicode/punycode equivalence, wildcard patterns, path regex, multi-subdomain wildcards, wildcard dot requirement |
| `TestStaticHTTPStatus` | Static HTTP status service returns configured status code |
| `TestMarshalJSON` | Rule serializes to JSON correctly |

### Key Behavioral Details

**Wildcard matching rules**:

| Pattern | Input | Matches? |
| --- | --- | --- |
| `*.example.com` | `foo.example.com` | Yes |
| `*.example.com` | `example.com` | No (dot required) |
| `*.example.com` | `foo.bar.example.com` | Yes (multi-subdomain) |
| `*.` | `foo.` | Yes |
| `*.*` | `foo.bar` | Yes |

**Punycode normalization**: The test validates that `münchen.de` matches rule `xn--mnchen-3ya.de` — hostname matching operates on punycode-normalized forms.

### Atom Links

- [ingress/rule](../../atoms/ingress/rule.md)

---

## ingress/origin\_proxy\_test.go

Source: [ingress/origin\_proxy\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/origin_proxy_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestRawTCPServiceEstablishConnection` | rawTCPService fails when origin not listening |
| `TestTCPOverWSServiceEstablishConnection` | Table-driven: specific TCP service connects; bastion connects via header; bastion without header fails; all fail when origin stops |
| `TestHTTPServiceHostHeaderOverride` | `HTTPHostHeader` config overrides request Host; `X-Forwarded-Host` preserves original |
| `TestHTTPServiceUsesIngressRuleScheme` | HTTP service uses ingress rule's URL scheme regardless of `X-Forwarded-Proto` header |

### Key Behavioral Details

**Bastion mode**: The bastion service extracts the destination from the `Cf-Access-Jump-Destination` header. Without this header, connection establishment fails. This is a critical contract for the Rust port's bastion-mode implementation.

**Host header override**: When `HTTPHostHeader` is configured, the proxy rewrites the `Host` header to the configured value and preserves the original in `X-Forwarded-Host`. This header rewriting contract must be preserved exactly.

### Atom Links

- [ingress/origin\_proxy](../../atoms/ingress/origin_proxy.md)

---

## ingress/origin\_icmp\_proxy\_test.go

Source: [ingress/origin\_icmp\_proxy\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/origin_icmp_proxy_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestICMPRouterEcho` | ICMP echo request/reply round-trip via ICMPRouter for both IPv4 and IPv6 |
| `TestTraceICMPRouterEcho` | Traced ICMP produces request span + reply + reply span; second request NOT traced |
| `TestConcurrentRequestsToSameDst` | 5 concurrent ping flows to same destination with different echo IDs all succeed |
| `TestICMPRouterRejectNotEcho` | Non-echo ICMP messages (DestinationUnreachable, TimeExceeded, PacketTooBig) rejected |

### Key Behavioral Details

**Tracing deduplication**: The first ICMP flow produces tracing spans (request + reply). Subsequent flows to the same destination are NOT traced — this prevents trace amplification under load.

**Concurrent flow isolation**: 5 simultaneous ICMP flows with different echo IDs to the same destination must all succeed independently. The test uses `leaktest.Check(t)()` to detect goroutine leaks.

### Mock Types

| Type | Purpose |
| --- | --- |
| `mockMuxer` | Channels for capturing packets sent from cfd to edge |

### Atom Links

- [ingress/origin\_icmp\_proxy](../../atoms/ingress/origin_icmp_proxy.md)

---

## ingress/origin\_connection\_test.go

Source: [ingress/origin\_connection\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/origin_connection_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestStreamTCPConnection` | TCP connection streams data bidirectionally between eyeball and origin with echo |
| `TestDefaultStreamWSOverTCPConnection` | WebSocket-over-TCP streams using DefaultStreamHandler |
| `TestSocksStreamWSOverTCPConnection` | SOCKS5 proxying wraps HTTP requests through WS-over-TCP across multiple status codes |
| `TestWsConnReturnsBeforeStreamReturns` | Stream returns cleanly after origin closes (no goroutine leak under 50 concurrent connections) |

### Mock Types

| Type | Purpose |
| --- | --- |
| `wsEyeball` | Wraps `net.Conn` with WebSocket read/write |
| `readWriter` | Simple I/O adapter |
| `echoWSOrigin` | WebSocket echo server via `httptest.NewServer` |
| `echoTCPOrigin` | TCP echo helper |

### Atom Links

- [ingress/origin\_connection](../../atoms/ingress/origin_connection.md)

---

## ingress/origin\_service\_test.go

Source: [ingress/origin\_service\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/origin_service_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestAddPortIfMissing` | Appends default SSH port (22) to URL host when port is absent |

### Atom Links

- [ingress/origin\_service](../../atoms/ingress/origin_service.md)

---

## ingress/config\_test.go

Source: [ingress/config\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/config_test.go)

Validates ingress configuration parsing and defaults.

### Atom Links

- [ingress/config](../../atoms/ingress/config.md)

---

## ingress/constants\_test.go

Source: [ingress/constants\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/constants_test.go)

Validates ingress constant definitions.

### Atom Links

- [ingress/constants](../../atoms/ingress/constants.md)

---

## ingress/packet\_router\_test.go

Source: [ingress/packet\_router\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/packet_router_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestRouterReturnTTLExceed` | Router replies with ICMP Time Exceeded for both IPv4 and IPv6 packets with TTL=1 |

### Mock Types

| Type | Purpose |
| --- | --- |
| `mockMuxer` | Channels for `cfdToEdge`/`edgeToCfd` with `SendPacket`/`ReceivePacket` |
| `routerEnabledChecker` | Atomic bool controlling router enable state |

### Atom Links

- [ingress/packet\_router](../../atoms/ingress/packet_router.md)

---

## ingress/icmp\_\*\_test.go (Platform-Specific)

Sources:

- [ingress/icmp\_darwin\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/icmp_darwin_test.go) — `//go:build darwin`
- [ingress/icmp\_linux\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/icmp_linux_test.go) — `//go:build linux`
- [ingress/icmp\_posix\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/icmp_posix_test.go) — `//go:build !windows`
- [ingress/icmp\_windows\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/icmp_windows_test.go) — `//go:build windows`

Platform-specific ICMP socket and proxy tests. These validate OS-dependent raw socket behavior (e.g., `IPPROTO_ICMP` vs `IPPROTO_RAW`, `IP_STRIPHDR` on macOS).

### Atom Links

- [ingress/icmp\_darwin](../../atoms/ingress/icmp_darwin.md)
- [ingress/icmp\_linux](../../atoms/ingress/icmp_linux.md)
- [ingress/icmp\_posix](../../atoms/ingress/icmp_posix.md)
- [ingress/icmp\_windows](../../atoms/ingress/icmp_windows.md)

---

## ingress/middleware/jwtvalidator\_test.go

Source: [ingress/middleware/jwtvalidator\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/middleware/jwtvalidator_test.go)

Validates JWT access-token verification middleware for ingress.

### Atom Links

- [ingress/middleware/jwtvalidator](../../atoms/ingress/middleware/jwtvalidator.md)

---

## ingress/origins/dns\_test.go & metrics\_test.go

Sources:

- [ingress/origins/dns\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/origins/dns_test.go)
- [ingress/origins/metrics\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/origins/metrics_test.go)

Validates origin DNS resolution and metrics collection.

### Atom Links

- [ingress/origins/dns](../../atoms/ingress/origins/dns.md)
- [ingress/origins/metrics](../../atoms/ingress/origins/metrics.md)

---

## orchestration/orchestrator\_test.go

Source: [orchestration/orchestrator\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/orchestration/orchestrator_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestUpdateConfiguration` | JSON config deserialization, proxy update, version check, noop for old version, invalid JSON handling |
| `TestUpdateConfiguration_FromMigration` | Version 0 for local→remote config migration |
| `TestUpdateConfiguration_WithoutIngressRule` | Empty config → default rule generated |
| `TestConcurrentUpdateAndRead` | 200 concurrent requests with 3 config versions; reads never see torn state |
| `TestOverrideWarpRoutingConfigWithLocalValues` | Local CLI flag (`MaxActiveFlows`) overrides remote config |
| `TestClosePreviousProxies` | Hello-world server shutdown on config change; re-spin on re-apply |
| `TestPersistentConnection` | WebSocket and TCP connections survive config updates |
| `TestSerializeLocalConfig` | Local config serializes to JSON correctly |

### Key Behavioral Details

**Concurrent config safety**: 200 goroutines simultaneously sending requests while 3 config updates are applied. The test validates that responses are always consistent with one of the valid config versions — no partial or torn reads.

**Persistent connections surviving config changes**: Active WebSocket and TCP connections established under config v1 continue to work after config v2 is applied. This is a critical contract: config hot-reload must not tear down in-flight connections.

**Hello-world server lifecycle**: Applying a config with `hello_world` service starts an HTTP server. Changing the config shuts it down. Re-applying the hello-world config starts a new server on the same port.

### Atom Links

- [orchestration/orchestrator](../../atoms/orchestration/orchestrator.md)

---

## orchestration/config\_test.go

Source: [orchestration/config\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/orchestration/config_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestNewLocalConfig_MarshalJSON` | LocalConfig JSON round-trip preserves ingress rules and warp-routing defaults |

### Atom Links

- [orchestration/config](../../atoms/orchestration/config.md)

---

## supervisor/tunnel\_test.go

Source: [supervisor/tunnel\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/supervisor/tunnel_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestWaitForBackoffFallback` | Protocol fallback state machine: QUIC→HTTP2 after max retries, reset behavior, `quic.IdleTimeoutError` immediate fallback, no-fallback-available exhaustion |

### Key Behavioral Details

**Protocol fallback state machine**: This is one of the most critical supervisor contracts. The fallback logic:

1. Starts on QUIC protocol
2. After `maxRetries` failures → falls back to HTTP2
3. `quic.IdleTimeoutError` triggers immediate fallback (no retry counting)
4. After `maxRetries` on HTTP2 → reset back to QUIC
5. When both protocols exhausted → returns "no fallback" error

The `BackoffHandler` with an injectable `Clock{Now, After}` controls retry timing.

### Atom Links

- [supervisor/tunnel](../../atoms/supervisor/tunnel.md)

---

## supervisor/pqtunnels\_test.go

Source: [supervisor/pqtunnels\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/supervisor/pqtunnels_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestCurvePreferences` | 8 cases: FIPS/non-FIPS × Prefer/Strict PQ × various current curves → resolved curve preference |
| `TestSupportedCurvesNegotiation` | Actual TLS handshake verifies that the selected curve preferences work end-to-end |

### Key Behavioral Details

**Post-quantum curve matrix**:

| Mode | Current Curve | FIPS | Expected |
| --- | --- | --- | --- |
| Prefer | P256 | No | [X25519Kyber768, P256] |
| Prefer | P256 | Yes | [P256Kyber768, P256] |
| Strict | P256 | No | [X25519Kyber768] |
| Strict | P256 | Yes | [P256Kyber768] |
| Prefer | X25519Kyber768 | No | [X25519Kyber768, P256] |
| Prefer | X25519Kyber768 | Yes | [P256Kyber768, P256] |
| Strict | — | No | [X25519Kyber768] |
| Strict | — | Yes | [P256Kyber768] |

### Atom Links

- [supervisor/pqtunnels](../../atoms/supervisor/pqtunnels.md)

---

## socks/request\_test.go

Source: [socks/request\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/socks/request_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestValidConnectRequest` | Well-formed SOCKS5 connect request (IPv4) parses successfully |
| `TestValidBindRequest` | Well-formed SOCKS5 bind request (IPv6) parses successfully |
| `TestValidAssociateRequest` | Well-formed SOCKS5 associate request parses successfully |
| `TestInValidVersionRequest` | SOCKS version 4 is rejected |
| `TestInValidIPRequest` | Malformed IP address is rejected |

### Key Behavioral Details

**Binary protocol construction**: The test manually builds SOCKS5 wire-format bytes (version byte, command, address type `1`=IPv4/`4`=IPv6, big-endian port encoding), providing a reference implementation for the Rust port's SOCKS5 parser.

### Atom Links

- [socks/request](../../atoms/socks/request.md)

---

## socks/request\_handler\_test.go

Source: [socks/request\_handler\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/socks/request_handler_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestUnsupportedBind` | BIND command → `commandNotSupported` |
| `TestUnsupportedAssociate` | ASSOCIATE command → `commandNotSupported` |
| `TestHandleConnect` | CONNECT to unreachable host → `connectionRefused` |
| `TestHandleConnectIPAccess` | IP access policies (allow/deny rules) enforced on CONNECT |

### Atom Links

- [socks/request\_handler](../../atoms/socks/request_handler.md)

---

## socks/connection\_handler\_test.go

Source: [socks/connection\_handler\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/socks/connection_handler_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestSocksConnection` | End-to-end integration: SOCKS5 proxy routes HTTP GET through to backend, returns valid JSON response |

### Atom Links

- [socks/connection\_handler](../../atoms/socks/connection_handler.md)

---

## carrier/carrier\_test.go

Source: [carrier/carrier\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/carrier/carrier_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestStartClient` | Client connects to WebSocket server, round-trips a message |
| `TestStartServer` | Server accepts TCP connection, proxies data through WebSocket, echoes back |
| `TestIsAccessResponse` | Identifies Cloudflare Access redirect responses (valid `/cdn-cgi/access/login/` location header) |
| `TestBastionDestination` | `ResolveBastionDest` extracts host:port from `Cf-Access-Jump-Destination` header; handles scheme stripping, paths, IPv4, IPv6, missing header error |

### Mock Types

| Type | Purpose |
| --- | --- |
| `testStreamer` | Locked `bytes.Buffer` implementing `io.ReadWriter` |

### Atom Links

- [carrier/carrier](../../atoms/carrier/carrier.md)

---

## carrier/websocket\_test.go

Source: [carrier/websocket\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/carrier/websocket_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestWebsocketHeaders` | websocketHeaders strips protocol-level WS headers but preserves application headers |
| `TestServe` | Full TLS WebSocket connection to hello-world server echoes 1000 random-sized binary messages |
| `TestWebsocketWrapper` | GorillaConn wrapper supports full and partial reads (1 byte + 2 bytes from 3-byte message) |

### Key Behavioral Details

**1000-message fuzz-like test**: Sends 1000 binary messages of random size (1–2048 bytes) over a TLS WebSocket to the hello-world server. Each message is echoed and compared byte-for-byte. This provides strong confidence in the WebSocket framing correctness.

### Atom Links

- [carrier/websocket](../../atoms/carrier/websocket.md)

---

## Contract Density Summary

| File | Tests | Benchmarks | Fuzz | Key Contracts |
| --- | --- | --- | --- | --- |
| proxy/proxy\_test.go | ~10 | 0 | 0 | 8 connection type permutations, SSE regression, error propagation |
| ingress/ingress\_test.go | ~11 | 1 | 0 | Rule parsing, service type mapping, YAML deserialization |
| ingress/rule\_test.go | 3 | 0 | 0 | Wildcard matching, punycode normalization |
| ingress/origin\_proxy\_test.go | 4 | 0 | 0 | Bastion mode, host header override |
| ingress/origin\_icmp\_proxy\_test.go | 4 | 0 | 0 | ICMP echo, trace dedup, concurrent flows |
| ingress/origin\_connection\_test.go | 4 | 0 | 0 | TCP/WS streaming, goroutine leak check |
| ingress/packet\_router\_test.go | 1 | 0 | 0 | TTL exceed response |
| orchestration/orchestrator\_test.go | 8 | 0 | 0 | Concurrent config safety, persistent connections, hello-world lifecycle |
| supervisor/tunnel\_test.go | 1 | 0 | 0 | Protocol fallback state machine |
| supervisor/pqtunnels\_test.go | 2 | 0 | 0 | PQ curve preference matrix |
| socks/request\_test.go | 5 | 0 | 0 | SOCKS5 wire format parsing |
| socks/request\_handler\_test.go | 4 | 0 | 0 | SOCKS command handling, IP access enforcement |
| socks/connection\_handler\_test.go | 1 | 0 | 0 | End-to-end SOCKS5 integration |
| carrier/carrier\_test.go | 4 | 0 | 0 | WebSocket proxying, bastion destination |
| carrier/websocket\_test.go | 3 | 0 | 0 | 1000-message TLS WebSocket echo |
| **Total** | **~65** | **1** | **0** | |
