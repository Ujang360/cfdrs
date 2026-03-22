# Tests — Transport Layer Contracts

- Parent: [tests](tests.md)
- Baseline reference: [cloudflare/cloudflared/tree/2026.3.0](https://github.com/cloudflare/cloudflared/tree/2026.3.0)

## Scope

This sub-catalog documents behavioral contracts encoded in transport-layer tests: connection establishment, QUIC server acceptance, HTTP/2 stream handling, header serialization, tunnel RPC, protocol selection, TLS configuration, and WebSocket fundamentals.

Packages covered: `connection`, `quic` (non-v3), `tunnelrpc`, `tlsconfig`, `websocket`.

---

## connection/connection\_test.go

Source: [connection/connection\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/connection/connection_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestShouldFlushHeaders` | Content-type determines header flushing: `text/event-stream`, `application/grpc`, `text/html; charset=utf-8` → flush; `application/json` → no flush |

### Mock Types

| Type | Purpose |
| --- | --- |
| `mockOriginProxy` | Configurable origin proxy returning static responses for HTTP, WebSocket, and TCP calls |
| `mockOrchestrator` | Returns `mockOriginProxy` for any ingress rule lookup |
| `mockConnectedFuse` | No-op connected/disconnected fuse for tunnel state |

### Test Infrastructure

| Helper | Purpose |
| --- | --- |
| `wsEchoEndpoint` | WebSocket endpoint that echoes messages back |
| `wsFlakyEndpoint` | WebSocket endpoint that randomly errors (for regression TUN-5184) |
| `originRespEndpoint` | HTTP endpoint returning static status and body |
| `testRequest` struct | Standard request builder with hostname, path, body, expected status |
| `largeFileSize = 2 * 1024 * 1024` | 2 MB payload constant for large-file tests |

### Atom Links

- [connection/connection](../../atoms/connection/connection.md)
- [connection/header](../../atoms/connection/header.md)

---

## connection/http2\_test.go

Source: [connection/http2\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/connection/http2_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestHTTP2ConfigurationSet` | Config update pushed via HTTP/2 PUT with JSON body; new version applied; old version rejected as no-op |
| `TestServeHTTP` | HTTP/2 server handles ok/large\_file/400/500/error endpoints with correct status codes and bodies |
| `TestServeWS` | WebSocket upgrade via HTTP/2 echoes messages; status 101 rewritten to 200 for HTTP/2 compliance |
| `TestNoWriteAfterServeHTTPReturns` | Regression TUN-5184: 100 concurrent flaky WS connections must not cause write-after-serve panic |
| `TestServeControlStream` | Registration via control stream → connection established; unregistration → disconnected |
| `TestFailRegistration` | Duplicate connection returns `errDuplicationConnection` |
| `TestGracefulShutdownHTTP2` | Sending shutdown signal causes graceful unregistration of the tunnel |
| `TestServeTCP_RateLimited` | Flow rate limiting propagates error + metadata header to edge |
| `BenchmarkServeHTTPSimple` | Performance baseline for simple HTTP/2 request serving |
| `BenchmarkServeHTTPLargeFile` | Performance baseline for 2 MB file serving over HTTP/2 |

### Key Behavioral Details

**Registration lifecycle**: The test validates the full control stream lifecycle: connect → registerConnection RPC → assert connection event → unregisterConnection → assert disconnection. This is the core tunnel registration contract.

**Write-after-serve regression (TUN-5184)**: Spawns 100 goroutines each opening a flaky WebSocket connection. The test passes by not panicking — it validates that the server correctly handles writer lifecycle even when connections close mid-stream.

**Config update via HTTP/2**: Uses a PUT request with `Content-Type: application/json` to push `UpdateConfigurationResponse`. The test validates version ordering — config with an older version is rejected as a no-op.

### Atom Links

- [connection/http2](../../atoms/connection/http2.md)
- [connection/control](../../atoms/connection/control.md)
- [connection/connection](../../atoms/connection/connection.md)

---

## connection/quic\_connection\_test.go

Source: [connection/quic\_connection\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/connection/quic_connection_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestQUICServer` | Table-driven: QUIC server correctly proxies HTTP (body echo), body streaming, WebSocket (echo), and TCP proxy types |
| `TestBuildHTTPRequest` | Content-length, chunked transfer-encoding, and WebSocket body handling are correctly set in built requests |
| `TestServeUDPSession` | V2 datagram sessions: closedByOrigin, closedByTimeout, and closedByRemote each produce correct close signals |
| `TestNopCloserReadWriterCloseBeforeEOF` | Close before EOF truncates reads correctly |
| `TestNopCloserReadWriterCloseAfterEOF` | Close after EOF is idempotent |
| `TestCreateUDPConnReuseSourcePort` | UDP source ports are cached per connIndex — same connIndex reuses port, different connIndex gets different port |
| `TestTCPProxy_FlowRateLimited` | Rate-limited TCP proxy returns `ConnectResponse` error with rate-limited metadata |

### Key Behavioral Details

**QUIC server multiplexing**: The table-driven test validates that a single QUIC server correctly discriminates between 4 proxy types (HTTP, streaming, WebSocket, TCP) using the same connection multiplexer. Each sub-test opens a QUIC stream, sends the appropriate request type, and validates the response.

**UDP port reuse**: Critical for performance — the test validates that `createUDPConnForConnIndex` returns the same port for the same connection index but different ports for different indices. The port is cached after first allocation.

**Session close semantics**: Three distinct close paths are tested:

1. **closedByOrigin** — origin server terminates → session closes with `closedByRemote=false`
2. **closedByTimeout** — idle timeout expires → session closes with idle error
3. **closedByRemote** — edge sends unregister → session closes with `closedByRemote=true`

### Mock Types

| Type | Purpose |
| --- | --- |
| `mockOriginProxyWithRequest` | Captures HTTP requests for assertion |
| `mockSessionRPCServer` | Configurable register/unregister UDP session handler |
| `fakeControlStream` | No-op control stream for QUIC server |
| `testTunnelConnection` | Helper composing QUIC server setup with default config |

### Atom Links

- [connection/quic\_connection](../../atoms/connection/quic_connection.md)
- [connection/quic](../../atoms/connection/quic.md)
- [connection/quic\_datagram\_v2](../../atoms/connection/quic_datagram_v2.md)

---

## connection/header\_test.go

Source: [connection/header\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/connection/header_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestSerializeHeaders` | HTTP headers round-trip through serialize → deserialize, preserving control characters, empty keys, special delimiters |
| `TestSerializeNoHeaders` | Empty header set serializes/deserializes to empty |
| `TestDeserializeMalformed` | Four malformed serialized headers (missing value, control chars in key, truncated) are rejected with errors |
| `TestIsControlResponseHeader` | Identifies `Cf-Transfer-Encoding` and `Cf-Cloudflared-Response-Meta` as control headers |
| `TestIsNotControlResponseHeader` | Regular headers (`Content-Type`, `x-custom`) are not classified as control headers |

### Key Behavioral Details

**Serialization format**: Headers are serialized as newline-delimited `key:value` pairs using a custom format (not standard HTTP). The test validates edge cases: control characters (ASCII 0-31), empty keys, colons in values, unicode characters. This is a critical wire-protocol contract for HTTP/2 tunnel framing.

### Atom Links

- [connection/header](../../atoms/connection/header.md)

---

## connection/observer\_test.go

Source: [connection/observer\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/connection/observer_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestSendUrl` | Observer URL notification increments a Prometheus counter for the hostname |
| `TestRegisterServerLocation` | Concurrent writes (20 goroutines) to server location map are safe |
| `TestObserverEventsDontBlock` | Sending more events than the channel buffer never blocks the caller |

### Key Behavioral Details

**Non-blocking event dispatch**: The observer uses a buffered channel for event distribution. The test validates that overflowing the buffer drops events silently rather than blocking — critical for tunnel stability under load.

### Mock Types

| Type | Purpose |
| --- | --- |
| `eventCollectorSink` | Collects events in a mutex-guarded slice |
| `EventSinkFunc` | Inline callback adapter to create a blocking sink |

### Atom Links

- [connection/observer](../../atoms/connection/observer.md)
- [connection/event](../../atoms/connection/event.md)

---

## connection/protocol\_test.go

Source: [connection/protocol\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/connection/protocol_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestNewProtocolSelector` | Table-driven: protocol selection respects user choice, auto-selects QUIC, forces PQ to QUIC, falls back to HTTP2, rejects unknown protocols |
| `TestAutoProtocolSelectorRefresh` | Auto-mode dynamically switches QUIC ↔ HTTP2 as remote percentage fetcher changes; sticks to last value on error |
| `TestHTTP2ProtocolSelectorRefresh` | Explicit HTTP2 selection never switches away, regardless of remote fetcher |
| `TestAutoProtocolSelectorNoRefreshWithToken` | With tunnel token, auto selector stays on QUIC and ignores percentage updates |

### Key Behavioral Details

**Protocol selection state machine**: The protocol selector implements a multi-factor decision:

1. **Explicit user choice** overrides everything
2. **Post-quantum mode** forces QUIC
3. **Tunnel token** forces QUIC (no auto-switch)
4. **Auto mode** uses remote percentage from DNS TXT record
5. **Fetch errors** preserve last-known-good value

This is important for the Rust port because the selector affects supervisor retry/fallback behavior.

### Mock Types

| Type | Purpose |
| --- | --- |
| `dynamicMockFetcher` | Mutable percentage + error for testing state transitions |
| `mockFetcher` function | Returns configurable `PercentageFetcher` closure |

### Atom Links

- [connection/protocol](../../atoms/connection/protocol.md)

---

## connection/quic\_datagram\_v2\_test.go

Source: [connection/quic\_datagram\_v2\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/connection/quic_datagram_v2_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestRateLimitOnNewDatagramV2UDPSession` | Flow limiter `ErrTooManyActiveFlows` rejects session registration; `Release()` is never called on failure |

### Key Behavioral Details

**Rate limiting precondition**: The test validates that when the flow limiter rejects a session, the datagram V2 connection does NOT call `Release()` — ensuring the flow count remains correct. gomock is used to assert exact call counts.

### Mock Types

| Type | Purpose |
| --- | --- |
| `mockQuicConnection` | Manual mock of full `quic.Connection` interface |
| `mocks.NewMockLimiter` | gomock-generated flow limiter mock |

### Atom Links

- [connection/quic\_datagram\_v2](../../atoms/connection/quic_datagram_v2.md)

---

## quic/datagram\_test.go

Source: [quic/datagram\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/quic/datagram_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestSuffixThenRemoveSessionID` | Session ID appended to / extracted from message payload round-trips correctly |
| `TestRemoveSessionIDError` | Too-short message rejects session ID extraction |
| `TestSuffixSessionIDError` | Max-sized payload accepts suffix; one byte over max fails |
| `TestDatagram` | End-to-end datagram mux/demux over real QUIC for v1 (sessions) and v2 (sessions + ICMP + tracing); oversized payloads rejected |

### Key Behavioral Details

**Real QUIC infrastructure**: Unlike most tests using mocks, this test creates actual QUIC client/server connections with self-signed TLS certificates. It validates the complete datagram path including serialization, QUIC transport, and deserialization.

**Version compatibility**: The test parametrizes on datagram version (v1 vs v2), validating that v1 carries only session payloads while v2 adds ICMP packet routing and tracing span transport.

### Atom Links

- [quic/datagram](../../atoms/quic/datagram.md)
- [quic/safe\_stream](../../atoms/quic/safe_stream.md)

---

## quic/safe\_stream\_test.go

Source: [quic/safe\_stream\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/quic/safe_stream_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestSafeStreamClose` | 1000 concurrent exchanges of 10 messages each over SafeStreamCloser-wrapped QUIC streams; half-closed streams must not break other streams in the session |

### Key Behavioral Details

**Concurrency stress test**: This is one of the most demanding tests in the suite. It validates that the `SafeStreamCloser` wrapper correctly handles concurrent stream operations without corruption or deadlock. 1000 goroutines each perform 10 round-trip message exchanges; even-numbered iterations perform a graceful half-close while odd-numbered iterations may fail gracefully.

### Atom Links

- [quic/safe\_stream](../../atoms/quic/safe_stream.md)

---

## tunnelrpc/pogs/registration\_server\_test.go

Source: [tunnelrpc/pogs/registration\_server\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/tunnelrpc/pogs/registration_server_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestMarshalConnectionOptions` | Cap'n Proto round-trip: marshal → unmarshal `ConnectionOptions` preserves all fields |
| `TestConnectionRegistrationRPC` | Full RPC flow: success returns `ConnectionDetails`; error propagates; `RetryableError` includes delay |

### Key Behavioral Details

**Cap'n Proto serialization fidelity**: The test validates that `ConnectionOptions` (including `Client`, `OriginLocalIP`, `ReplaceExisting`, `CompressionQuality`, `NumPreviousAttempts`) survive a Cap'n Proto marshal/unmarshal cycle. This is critical for the Rust port because Cap'n Proto schema compatibility must be maintained exactly.

**Registration error taxonomy**: Three scenarios are tested sequentially on the same RPC connection:

1. Success → returns `ConnectionDetails` with UUID and location
2. Error → propagates error string
3. RetryableError → propagates error + `RetryAfter` delay

### Mock Types

| Type | Purpose |
| --- | --- |
| `testConnectionRegistrationServer` | Configurable `RegisterConnection`/`UnregisterConnection` returning preset details/errors |

### Atom Links

- [tunnelrpc/pogs/registration\_server](../../atoms/tunnelrpc/pogs/registration_server.md)

---

## tunnelrpc/quic/request\_server\_stream\_test.go

Source: [tunnelrpc/quic/request\_server\_stream\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/tunnelrpc/quic/request_server_stream_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestConnectRequestData` | Write + read connect request preserves hostname, connection type, and metadata |
| `TestConnectResponseMeta` | Write + read connect response preserves metadata; error clears other fields |
| `TestRegisterUdpSession` | Full RPC: register/unregister UDP sessions; wrong sessionID rejected; with/without trace context |
| `TestManageConfiguration` | `UpdateConfiguration` RPC returns correct `LastAppliedVersion` |

### Mock Types

| Type | Purpose |
| --- | --- |
| `mockSessionRPCServer` | Validates sessionID, IP, port, trace context fields |
| `mockConfigRPCServer` | Validates version and config bytes |
| `mockRPCStream` | `io.Pipe` pair simulating bidirectional stream |
| `noopCloser` | Wraps ReadWriter with no-op Close |

### Atom Links

- [tunnelrpc/quic/request\_server\_stream](../../atoms/tunnelrpc/quic/request_server_stream.md)

---

## tlsconfig/tlsconfig\_test.go

Source: [tlsconfig/tlsconfig\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/tlsconfig/tlsconfig_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestGetFromEmptyConfig` | Empty TLS parameters yield config with no certs, no CAs, CurveP256 default |
| `TestGetConfig` | Full parameters produce matching certs, client/root CA pools, and custom curve |
| `TestCertReloader` | CertReloader dynamically returns correct cert via `GetCertificate` callback |

### Atom Links

- [tlsconfig/tlsconfig](../../atoms/tlsconfig/tlsconfig.md)

---

## websocket/websocket\_test.go

Source: [websocket/websocket\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/websocket/websocket_test.go)

### Contracts

| Test | Behavioral Contract |
| --- | --- |
| `TestGenerateAcceptKey` | RFC 6455 `Sec-WebSocket-Accept` key generation produces correct value from standard test vector |

### Key Behavioral Details

Uses the RFC 6455 test vector: key `dGhlIHNhbXBsZSBub25jZQ==` must produce accept `s3pPLMBiTxaQ9kYGzzhZRbK+xOo=`. Single assertion, but critical for WebSocket handshake compliance.

### Atom Links

- [websocket/websocket](../../atoms/websocket/websocket.md)

---

## Contract Density Summary

| File | Tests | Benchmarks | Fuzz | Key Contracts |
| --- | --- | --- | --- | --- |
| connection/connection\_test.go | 1 | 0 | 0 | Header flush decision |
| connection/http2\_test.go | 7 | 2 | 0 | Registration lifecycle, config update, graceful shutdown, TUN-5184 regression |
| connection/quic\_connection\_test.go | 7 | 0 | 0 | 4 proxy types via QUIC, UDP port reuse, session close semantics |
| connection/header\_test.go | 5 | 0 | 0 | Header serialization round-trip, malformed rejection |
| connection/observer\_test.go | 3 | 0 | 0 | Non-blocking event dispatch, concurrent safety |
| connection/protocol\_test.go | 4 | 0 | 0 | Protocol selection state machine |
| connection/quic\_datagram\_v2\_test.go | 1 | 0 | 0 | Rate-limit rejection without Release |
| quic/datagram\_test.go | 4 | 0 | 0 | Real QUIC datagram mux/demux, version compat |
| quic/safe\_stream\_test.go | 1 | 0 | 0 | 1000-goroutine stream concurrency |
| tunnelrpc/pogs/registration\_server\_test.go | 2 | 0 | 0 | Cap'n Proto round-trip, registration error taxonomy |
| tunnelrpc/quic/request\_server\_stream\_test.go | 4 | 0 | 0 | Connect request/response, UDP session RPC, config RPC |
| tlsconfig/tlsconfig\_test.go | 3 | 0 | 0 | TLS config building, cert reloading |
| websocket/websocket\_test.go | 1 | 0 | 0 | RFC 6455 accept key |
| **Total** | **43** | **2** | **0** | |
