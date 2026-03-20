# Tests — Infrastructure & Support Contracts

- Parent: [tests](tests.md)
- Baseline reference: [cloudflare/cloudflared/tree/2026.3.0](https://github.com/cloudflare/cloudflared/tree/2026.3.0)

## Scope

This sub-catalog documents behavioral contracts encoded in management service, diagnostics, edge discovery, cfapi, credentials, tokens, features, config, metrics, validation, logging, signals, file watching, overwatch, hello-world, and other support package tests.

Packages covered: `management`, `diagnostic`, `edgediscovery`, `cfapi`, `credentials`, `token`, `features`, `config`, `client`, `metrics`, `validation`, `logger`, `signal`, `watcher`, `retry`, `overwatch`, `hello`, `sshgen`, `ipaccess`, `tracing`, `cmd`.

---

## management/service\_test.go

Source: [management/service\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/management/service_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestDisableDiagnosticRoutes` | Metrics and pprof routes disabled when diagnostic disabled |
| `TestReadEventsLoop` | WebSocket event streaming loop delivers log events to connected client |
| `TestReadEventsLoop_ContextCancelled` | Context cancellation stops the event loop cleanly |
| `TestCanStartStream_NoSessions` | New stream allowed when no sessions exist |
| `TestCanStartStream_ExistingSessionDifferentActor` | Session limit (1) blocks different actor |
| `TestCanStartStream_ExistingSessionSameActor` | Same actor replaces existing session |

### Key Behavioral Details

**Session limit enforcement**: The management service allows at most 1 active streaming session. A second actor is rejected. The same actor can reconnect and replace their existing session.

### Atom Links

- [management/service](../../atoms/management/service.md)

---

## management/events\_test.go

Source: [management/events\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/management/events_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestIntoClientEvent_StartStreaming` | Deserializes `EventStartStreaming` with filter combinations (none, level, events, sampling, combined) |
| `TestIntoClientEvent_StopStreaming` | Deserializes `EventStopStreaming` |
| `TestIntoClientEvent_Invalid` | Unknown/mismatched event type returns false |
| `TestIntoServerEvent_Logs` | Deserializes `EventLog` |
| `TestIntoServerEvent_Invalid` | Unknown event type returns false |
| `TestReadServerEvent` | WebSocket write → read round-trip for server events |
| `TestReadServerEvent_InvalidWebSocketMessageType` | Binary WS message (instead of text) causes error |
| `TestReadServerEvent_InvalidMessageType` | Client event type sent as server event causes `errInvalidMessageType` |
| `TestReadClientEvent` | WebSocket write → read round-trip for client events |
| `TestReadClientEvent_InvalidWebSocketMessageType` | Binary WS message causes error |
| `TestReadClientEvent_InvalidMessageType` | Unknown client event type causes error |

### Key Behavioral Details

**WebSocket event protocol**: Management events are JSON-encoded text WebSocket messages. The tests validate:

- Client → Server: `StartStreaming` (with filters), `StopStreaming`
- Server → Client: `Logs` events
- Binary WebSocket messages are rejected (must be text)
- Event type mismatches are detected

### Atom Links

- [management/events](../../atoms/management/events.md)

---

## management/logger\_test.go

Source: [management/logger\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/management/logger_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestLoggerWrite_NoSessions` | Writing with no listeners does not panic or block |
| `TestLoggerWrite_OneSession` | Single session receives log event with correct Time, Message, Level, Event fields |
| `TestLoggerWrite_MultipleSessions` | All sessions receive same event; removed session receives nothing |
| `TestParseZerologEvent_EventTypes` | All `LogEventType` values (Cloudflared, HTTP, TCP, UDP) parsed correctly; invalid defaults to Cloudflared |
| `TestParseZerologEvent_Fields` | Top-level zerolog keys excluded from `Fields` map; custom fields preserved |

### Atom Links

- [management/logger](../../atoms/management/logger.md)

---

## management/middleware\_test.go

Source: [management/middleware\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/management/middleware_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestValidateAccessTokenQueryMiddleware` | Valid `access_token` query param → 200 with claims; missing → 400; malformed → 400 |

### Atom Links

- [management/middleware](../../atoms/management/middleware.md)

---

## management/session\_test.go

Source: [management/session\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/management/session_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestSession_ActiveControl` | Session starts inactive; activatable via atomic store; `Stop()` deactivates |
| `TestSession_Insert` | Filters by level, event type, and sampling rate; correctly passes or drops events |
| `TestSession_InsertOverflow` | Full listener channel (capacity=1) drops excess inserts silently (non-blocking) |

### Key Behavioral Details

**Streaming filters**: The session supports three filter dimensions:

| Filter | Behavior |
|---|---|
| Level (e.g., Warn) | Events below configured level are dropped |
| Events (e.g., HTTP only) | Non-matching event types dropped |
| Sampling (0.0–1.0) | Random sampling based on configurable rate |

### Atom Links

- [management/session](../../atoms/management/session.md)

---

## management/token\_test.go

Source: [management/token\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/management/token_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestParseToken` | Table-driven: valid claims accepted; missing tunnel, tunnel ID, account tag, actor, actor ID, and invalid schema all rejected |

### Key Behavioral Details

Signs test JWTs with ECDSA P-256 using `go-jose/v4`. Validates claim structure: `Tunnel{Id, AccountTag}`, `Actor{Id}`, issuer `"tunnelstore"`.

### Atom Links

- [management/token](../../atoms/management/token.md)

---

## diagnostic/handlers\_test.go

Source: [diagnostic/handlers\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/diagnostic/handlers_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestSystemHandler` | Returns 200 with system info; returns 200 (graceful) even on collector error |
| `TestTunnelStateHandler` | Serializes tunnel ID, connector ID, connections, ICMP sources |
| `TestConfigurationHandler` | Returns CLI flags as JSON; `uid` key always injected |

### Mock Types

| Type | Purpose |
|---|---|
| `SystemCollectorMock` | Configurable system info and error |

### Atom Links

- [diagnostic/handlers](../../atoms/diagnostic/handlers.md)

---

## diagnostic/diagnostic\_utils\_test.go

Source: [diagnostic/diagnostic\_utils\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/diagnostic/diagnostic_utils_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestFindMetricsServer_WhenSingleServerIsRunning_ReturnState` | Single metrics server → return state |
| `TestFindMetricsServer_WhenMultipleServerAreRunning_ReturnError` | Multiple servers → `ErrMultipleMetricsServerFound` |
| `TestFindMetricsServer_WhenNoInstanceIsRuning_ReturnError` | No server → `ErrMetricsServerNotFound` |

### Atom Links

- [diagnostic/diagnostic\_utils](../../atoms/diagnostic/diagnostic_utils.md)

---

## diagnostic/system\_collector\_test.go

Source: [diagnostic/system\_collector\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/diagnostic/system_collector_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestParseMemoryInformationFromKV` | Parses Linux, macOS, and Windows memory KV output |
| `TestParseUnameOutput` | Parses `uname -a` for Darwin and Linux |
| `TestParseFileDescriptorInformationFromKV` | Parses macOS sysctl KV |
| `TestParseSysctlFileDescriptorInformation` | Parses Linux `/proc/sys/fs/file-nr` |
| `TestParseWinOperatingSystemInfo` | Parses Windows PowerShell output |
| `TestParseDiskVolumeInformationOutput` | Parses Unix `df` and Windows volume output |

### Key Behavioral Details

**Multi-platform parsing**: Each test covers 2–3 OS-specific output formats. The Rust port must implement platform-specific parsers for the same output formats or use native system APIs.

### Atom Links

- [diagnostic/system\_collector](../../atoms/diagnostic/system_collector.md)

---

## edgediscovery/edgediscovery\_test.go

Source: [edgediscovery/edgediscovery\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/edgediscovery/edgediscovery_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestGiveBack` | Address returned to pool when connection fails |
| `TestRPCAndProxyShareSingleEdgeIP` | Single edge IP usable for both tunnel and RPC simultaneously |
| `TestGetAddrForRPC` | RPC address usage does not consume from available pool |
| `TestOnePerRegion` | Address rotation on failure; return to first on second failure |
| `TestOnlyOneAddrLeft` | Single address: failure → error; next iteration → re-available |
| `TestNoAddrsLeft` | Empty edge → error for both tunnel and RPC |
| `TestGetAddr` | Same connID gets same address on repeated calls |
| `TestGetDifferentAddr` | Different address requested for same connID on failure |

### Key Behavioral Details

**Edge address lifecycle**: The edge discovery system maintains a pool of Cloudflare edge addresses. Each tunnel connection gets a unique address. On connection failure, the address is returned to the pool. RPC requests can share addresses with tunnel connections.

### Atom Links

- [edgediscovery/edgediscovery](../../atoms/edgediscovery/edgediscovery.md)

---

## edgediscovery/allregions/\*\_test.go

Sources:

- [allregions/address\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/edgediscovery/allregions/address_test.go) — `AddrUsedBy`, `AvailableAddrs`, `GetUnusedIP`, `GiveBack`, `GetAnyAddress`
- [allregions/region\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/edgediscovery/allregions/region_test.go) — Region initialization for IPv4Only/IPv6Only/Auto, primary/secondary failover, GiveBack with/without connectivity error, timeout-based swap
- [allregions/discovery\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/edgediscovery/allregions/discovery_test.go) — DNS lookup mock: `mockNetLookupSRV`, `mockNetLookupIP`
- [allregions/regions\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/edgediscovery/allregions/regions_test.go) — Multi-region address pool management

### Key Behavioral Details

**IPv4/IPv6 failover**: The region system maintains primary (IPv6) and secondary (IPv4) address sets. When IPv6 connectivity fails, it swaps to IPv4. When IPv4 also fails, it swaps back. A timeout prevents rapid oscillation between address families.

### Atom Links

- [edgediscovery/allregions/address](../../atoms/edgediscovery/allregions/address.md)
- [edgediscovery/allregions/region](../../atoms/edgediscovery/allregions/region.md)
- [edgediscovery/allregions/discovery](../../atoms/edgediscovery/allregions/discovery.md)
- [edgediscovery/allregions/regions](../../atoms/edgediscovery/allregions/regions.md)

---

## edgediscovery/protocol\_test.go

Source: [edgediscovery/protocol\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/edgediscovery/protocol_test.go)

Edge discovery protocol-level tests.

### Atom Links

- [edgediscovery/protocol](../../atoms/edgediscovery/protocol.md)

---

## cfapi/hostname\_test.go

Source: [cfapi/hostname\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/cfapi/hostname_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestDNSRouteUnmarshalResult` | DNS route JSON deserialization (success + malformed/error rejection) |
| `TestLBRouteUnmarshalResult` | LB route JSON deserialization with pool + load\_balancer changes |
| `TestLBRouteResultSuccessSummary` | 3×3 Change enum matrix → human-readable summary strings |

### Atom Links

- [cfapi/hostname](../../atoms/cfapi/hostname.md)

---

## cfapi/ip\_route\_test.go

Source: [cfapi/ip\_route\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/cfapi/ip_route_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestUnmarshalRoute` | Route JSON deserialization with/without VNet ID |
| `TestDetailedRouteJsonRoundtrip` | JSON unmarshal → marshal round-trip produces identical output |
| `TestMarshalNewRoute` | NewRoute serializes with `tunnel_id`; VNet ID omitted when nil |
| `TestRouteTableString` | Tab-delimited row formatting |

### Atom Links

- [cfapi/ip\_route](../../atoms/cfapi/ip_route.md)

---

## cfapi/tunnel\_test.go

Source: [cfapi/tunnel\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/cfapi/tunnel_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `Test_unmarshalTunnel` | Table-driven: tunnel JSON with `deleted_at`, `conns_active_at=null` |
| `TestUnmarshalTunnelOk` | Minimal valid tunnel JSON |
| `TestUnmarshalTunnelErr` | Malformed/error-response JSONs produce error |
| `TestManagementResource_String` | Enum → lowercase string mapping |
| `TestManagementResource_String_Unknown` | Unknown enum → empty string |
| `TestManagementEndpointPath` | URL path construction: `{tunnelID}/management/{resource}` |
| `TestUnmarshalConnections` | Active-client JSON with nested connections |

### Atom Links

- [cfapi/tunnel](../../atoms/cfapi/tunnel.md)

---

## cfapi/virtual\_network\_test.go

Source: [cfapi/virtual\_network\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/cfapi/virtual_network_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestVirtualNetworkJsonRoundtrip` | JSON unmarshal → marshal byte-identical |
| `TestMarshalNewVnet` | Serializes name correctly |
| `TestMarshalUpdateVnet` | Partial field serialization |
| `TestVnetTableString` | Tab-delimited formatting |

### Atom Links

- [cfapi/virtual\_network](../../atoms/cfapi/virtual_network.md)

---

## credentials/credentials\_test.go

Source: [credentials/credentials\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/credentials/credentials_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestCredentialsRead` | Cert file → User with correct certPath, APIToken, ZoneID, AccountID |
| `TestCredentialsClient` | Valid cert → non-nil API client |

### Atom Links

- [credentials/credentials](../../atoms/credentials/credentials.md)

---

## credentials/origin\_cert\_test.go

Source: [credentials/origin\_cert\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/credentials/origin_cert_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestLoadOriginCert` | Empty cert → error; unknown PEM block → error with block name |
| `TestJSONArgoTunnelTokenEmpty` | Cert with no token → "missing token" error |
| `TestJSONArgoTunnelToken` | Valid JSON tunnel token decodes correctly (zoneID, apiToken) |
| `TestFindOriginCert_Valid` | Existing file → cert path without error |
| `TestFindOriginCert_Missing` | Missing file → error |
| `TestEncodeDecodeOriginCert` | Round-trip encode→decode preserves ZoneID, AccountID, APIToken, Endpoint |
| `TestEncodeDecodeNilOriginCert` | Nil cert → encode error |

### Atom Links

- [credentials/origin\_cert](../../atoms/credentials/origin_cert.md)

---

## token/token\_test.go

Source: [token/token\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/token/token_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestHandleRedirects_AttachOrgToken` | Redirect handler attaches org token cookie to login request |
| `TestHandleRedirects_AttachAppSessionCookie` | Redirect handler forwards `CF_AppSession` cookie from via-chain |
| `TestHandleRedirects_StopAtAuthorizedEndpoint` | Returns `http.ErrUseLastResponse` at authorized worker path |
| `TestJwtPayloadUnmarshal_AudAsString` | JWT `aud` as string → single-element slice |
| `TestJwtPayloadUnmarshal_AudAsSlice` | JWT `aud` as string array → multi-element slice |
| `TestJwtPayloadUnmarshal_FailsWhenAudIsInt` | JWT `aud` as integer → error |
| `TestJwtPayloadUnmarshal_FailsWhenAudIsArrayOfInts` | JWT `aud` as int array → "non-string elements" error |
| `TestJwtPayloadUnmarshal_FailsWhenAudIsOmitted` | Missing JWT `aud` → error |

### Key Behavioral Details

**JWT `aud` field polymorphism**: The `aud` (audience) claim can be either a string or an array of strings per RFC 7519. The test validates that both forms are accepted and that non-string values are rejected with descriptive errors.

### Atom Links

- [token/token](../../atoms/token/token.md)

---

## token/signal\_test.go

Source: [token/signal\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/token/signal_test.go)

Build constraint: `//go:build linux || darwin`

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestSignalHandler` | SIGUSR1 delivery triggers registered callback |
| `TestSignalHandlerClose` | Deregistered handler prevents callback execution |

### Atom Links

- [token/signal](../../atoms/token/signal.md)

---

## features/selector\_test.go

Source: [features/selector\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/features/selector_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestUnmarshalFeaturesRecord` | Feature record JSON with `dv3_2` percentage field (0, 39, 100, missing, unknown keys) |
| `TestFeaturePrecedenceEvaluationPostQuantum` | Default → PostQuantumPrefer; CLI flag → PostQuantumStrict |
| `TestFeaturePrecedenceEvaluationDatagramVersion` | Default → DatagramV2; CLI `FeatureDatagramV3_2` → DatagramV3 |
| `TestDeprecatedFeaturesRemoved` | Deprecated features (`DatagramV3`, `DatagramV3_1`) are stripped from feature list |
| `TestRefreshFeaturesRecord` | Percentage threshold determines v2/v3 activation; resolver error preserves last value |
| `TestSnapshotIsolation` | Snapshots are immutable after creation; new refresh produces different snapshot |
| `TestStaticFeatures` | PQ flag from CLI → PostQuantumStrict; no flag → PostQuantumPrefer |

### Key Behavioral Details

**Feature flag evaluation**: The feature selector uses DNS TXT record lookup to fetch a percentage. If `accountHash < percentage`, the feature is enabled. The test validates:

- Percentage 0 → v2 (below threshold)
- Percentage > hash → v3 (above threshold)
- Resolver error → preserve last known value
- Deprecated features stripped from feature list

### Mock Types

| Type | Purpose |
|---|---|
| `mockResolver` | Returns configurable percentages sequentially |
| `staticResolver` | Returns fixed `featuresRecord` |

### Atom Links

- [features/selector](../../atoms/features/selector.md)

---

## config/configuration\_test.go

Source: [config/configuration\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/config/configuration_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestConfigFileSettings` | YAML config deserializes to correct tunnel ID, ingress rules, warp-routing, IP rules, typed scalar accessors |
| `TestMarshalUnmarshalOriginRequest` | `OriginRequestConfig` round-trips through JSON and YAML |

### Atom Links

- [config/configuration](../../atoms/config/configuration.md)

---

## config/manager\_test.go

Source: [config/manager\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/config/manager_test.go)

Config manager tests.

### Atom Links

- [config/manager](../../atoms/config/manager.md)

---

## client/config\_test.go

Source: [client/config\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/client/config_test.go)

Client configuration tests.

### Atom Links

- [client/config](../../atoms/client/config.md)

---

## metrics/metrics\_test.go

Source: [metrics/metrics\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/metrics/metrics_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestMetricsListenerCreation` | 5 listeners on sequential ports (20241–20245); 6th gets different port; custom address respected |

### Atom Links

- [metrics/metrics](../../atoms/metrics/metrics.md)

---

## metrics/readiness\_test.go

Source: [metrics/readiness\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/metrics/readiness_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestReadinessEventHandling` | Full lifecycle: not-ready → connected (200) → multiple connected → reconnecting (still OK) → registering (not OK) → connected → unregistering (not OK) → disconnected (not OK) |

### Key Behavioral Details

**Readiness state machine**: The readiness endpoint transitions through tunnel connection states. Key invariant: with at least one healthy connection, the endpoint returns 200 even if another connection is reconnecting. Regression reference: TUN-3777.

### Atom Links

- [metrics/readiness](../../atoms/metrics/readiness.md)

---

## tracing/tracing\_test.go

Source: [tracing/tracing\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/tracing/tracing_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestNewCfTracer` | Valid `Cf-Access-Trace-Id` header → real TracerProvider with InMemoryOtlpClient |
| `TestNewCfTracerMultiple` | Multiple trace headers → valid tracer (takes first) |
| `TestNewCfTracerNilHeader` | Nil header → NoopOtlpClient |
| `TestNewCfTracerInvalidHeaders` | Empty/nil headers → noop tracer |
| `TestAddingSpansWithNilMap` | `AddSpans(nil)` does not panic |
| `FuzzNewIdentity` | Fuzz: arbitrary strings → no panic on identity parsing |

### Atom Links

- [tracing/tracing](../../atoms/tracing/tracing.md)

---

## tracing/identity\_test.go

Source: [tracing/identity\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/tracing/identity_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestNewIdentity` | Table-driven: full-length, short (zero-padded), empty, malformed traces; round-trip via MarshalBinary/UnmarshalBinary |

### Atom Links

- [tracing/identity](../../atoms/tracing/identity.md)

---

## tracing/client\_test.go

Source: [tracing/client\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/tracing/client_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestUploadTraces` | InMemoryOtlpClient accumulates spans across calls |
| `TestSpans` | Spans() returns base64-encoded protobuf |
| `TestSpansEmpty` | Empty upload → `errNoTraces` |
| `TestSpansNil` | Nil upload → `errNoTraces` |
| `TestSpansTooManySpans` | Client caps at `MaxTraceAmount` but still succeeds |

### Key Behavioral Details

**Interface compliance assertions**:

```go
var _ otlptrace.Client = (*InMemoryOtlpClient)(nil)
var _ InMemoryClient = (*InMemoryOtlpClient)(nil)
var _ otlptrace.Client = (*NoopOtlpClient)(nil)
var _ InMemoryClient = (*NoopOtlpClient)(nil)
```

### Atom Links

- [tracing/client](../../atoms/tracing/client.md)

---

## validation/validation\_test.go

Source: [validation/validation\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/validation/validation_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestValidateHostname` | Hostname normalization: empty, plain, URL-prefixed, IDN punycode, percent-encoded |
| `TestValidateUrl` | URL normalization: scheme inference, path/port, IDN, IPv4, IPv6, credential stripping, ftp rejection |
| `TestNewAccessValidatorOk` | Access validator creation for valid domain; rejects empty/invalid JWT |
| `TestNewAccessValidatorErr` | Rejects empty, ftp://, wss:// URLs |
| `FuzzNewAccessValidator` | Fuzz: arbitrary inputs → no panic |

### Key Behavioral Details

**IDN punycode**: `bücher.example.com` → `xn--bcher-kva.example.com` — hostname matching operates on punycode-normalized forms.

### Atom Links

- [validation/validation](../../atoms/validation/validation.md)

---

## ipaccess/access\_test.go

Source: [ipaccess/access\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ipaccess/access_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestRuleCreation` | Null ipnet, out-of-range ports rejected; valid ports sorted |
| `TestRuleCreationByCIDR` | CIDR string parsing: nil, bad, good |
| `TestRulesNoRules` | Empty policy: allow=true allows, allow=false denies |
| `TestRulesMatchIPAndPort` | Policy matches IP+port against rules |
| `TestRulesMatchIPAndPort2` | Deny rule precedence; unmatched port denied |

### Atom Links

- [ipaccess/access](../../atoms/ipaccess/access.md)

---

## signal/safe\_signal\_test.go

Source: [signal/safe\_signal\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/signal/safe_signal_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestMultiNotifyDoesntCrash` | Double Notify does not panic (safe for double-close) |
| `TestWait` | After Notify, Wait channel is immediately readable |

### Atom Links

- [signal/safe\_signal](../../atoms/signal/safe_signal.md)

---

## watcher/file\_test.go

Source: [watcher/file\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/watcher/file_test.go)

Build constraint: `//go:build !windows`

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestFileChanged` | File watcher detects write to watched file and notifies callback |

### Mock Types

| Type | Purpose |
|---|---|
| `mockNotifier` | Captures event path from WatcherItemDidChange |

### Atom Links

- [watcher/file](../../atoms/watcher/file.md)

---

## retry/backoffhandler\_test.go

Source: [retry/backoffhandler\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/retry/backoffhandler_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestBackoffRetries` | Allows exactly maxRetries attempts, then refuses |
| `TestBackoffCancel` | Respects context cancellation |
| `TestBackoffGracePeriod` | Grace period expiry resets retry counter |
| `TestGetMaxBackoffDurationRetries` | GetMaxBackoffDuration tracks same as Backoff |
| `TestGetMaxBackoffDuration` | Exponential durations: ≤2s, ≤4s, ≤8s, then 0 at exhaustion |
| `TestBackoffRetryForever` | retryForever=true → never refuses; durations cap at ≤16s |

### Key Behavioral Details

**Exponential backoff**: Durations grow exponentially: 2s → 4s → 8s → 16s → cap. The test uses an injectable `Clock{Now, After}` for deterministic timing.

### Atom Links

- [retry/backoffhandler](../../atoms/retry/backoffhandler.md)

---

## overwatch/manager\_test.go

Source: [overwatch/manager\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/overwatch/manager_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestManagerAddAndRemove` | Adds 2 services, removes 1 by name |
| `TestManagerDuplicate` | Adding same service twice results in 1 service |
| `TestManagerErrorChannel` | Service run error propagated through manager callback |

### Mock Types

| Type | Purpose |
|---|---|
| `mockService` | Configurable Name/Type/Hash/Run/Shutdown with optional error |

### Atom Links

- [overwatch/manager](../../atoms/overwatch/manager.md)

---

## hello/hello\_test.go

Source: [hello/hello\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/hello/hello_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestCreateTLSListenerHostAndPortSuccess` | `CreateTLSListener("localhost:1234")` succeeds |
| `TestCreateTLSListenerOnlyHostSuccess` | `CreateTLSListener("localhost:")` succeeds with OS-assigned port |
| `TestCreateTLSListenerOnlyPortSuccess` | `CreateTLSListener("localhost:8888")` succeeds |

### Atom Links

- [hello/hello](../../atoms/hello/hello.md)

---

## logger/create\_test.go

Source: [logger/create\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/logger/create_test.go)

### Contracts

| Test | Behavioral Contract |
|---|---|
| `TestResilientMultiWriter_Errors` | All writers receive writes regardless of individual writer errors |
| `TestResilientMultiWriter_Management` | Management writer receives WriteLevel calls for all levels |

### Mock Types

| Type | Purpose |
|---|---|
| `mockedWriter` | Configurable error, counts write calls |
| `mockedManagementWriter` | Counts WriteLevel calls |

### Atom Links

- [logger/create](../../atoms/logger/create.md)

---

## cmd/\* Tests

Sources:

- [cmd/cloudflared/access/validation\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/cmd/cloudflared/access/validation_test.go) — Access CLI validation
- [cmd/cloudflared/management/cmd\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/cmd/cloudflared/management/cmd_test.go) — Management CLI command tests
- [cmd/cloudflared/tunnel/cmd\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/cmd/cloudflared/tunnel/cmd_test.go) — Tunnel CLI command tests
- [cmd/cloudflared/tunnel/cmd\_test/configuration\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/cmd/cloudflared/tunnel/cmd_test/configuration_test.go) — Tunnel config integration tests
- [cmd/cloudflared/tunnel/signal\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/cmd/cloudflared/tunnel/signal_test.go) — Tunnel signal handling
- [cmd/cloudflared/tunnel/subcommand\_context\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/cmd/cloudflared/tunnel/subcommand_context_test.go) — Subcommand context tests
- [cmd/cloudflared/tunnel/subcommands\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/cmd/cloudflared/tunnel/subcommands_test.go) — Tunnel subcommand tests
- [cmd/cloudflared/tunnel/tag\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/cmd/cloudflared/tunnel/tag_test.go) — Tag parsing tests
- [cmd/cloudflared/updater/update\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/cmd/cloudflared/updater/update_test.go) — Auto-update tests
- [cmd/cloudflared/updater/workers\_service\_test.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/cmd/cloudflared/updater/workers_service_test.go) — Workers update service

### Atom Links

- [cmd/cloudflared/tunnel/cmd](../../atoms/cmd/cloudflared/tunnel/cmd.md)
- [cmd/cloudflared/tunnel/configuration](../../atoms/cmd/cloudflared/tunnel/configuration.md)
- [cmd/cloudflared/updater/update](../../atoms/cmd/cloudflared/updater/update.md)

---

## Contract Density Summary

| File | Tests | Fuzz | Key Contracts |
|---|---|---|---|
| management/service\_test.go | 6 | 0 | Session limits, diagnostic routes |
| management/events\_test.go | 11 | 0 | WebSocket event protocol |
| management/logger\_test.go | 5 | 0 | Multi-session log distribution |
| management/session\_test.go | 3 | 0 | Streaming filters |
| management/token\_test.go | 1 | 0 | JWT claim validation |
| management/middleware\_test.go | 1 | 0 | Access token middleware |
| diagnostic/handlers\_test.go | 3 | 0 | HTTP diagnostic endpoints |
| diagnostic/diagnostic\_utils\_test.go | 3 | 0 | Metrics server discovery |
| diagnostic/system\_collector\_test.go | 6 | 0 | Multi-platform parsing |
| edgediscovery/edgediscovery\_test.go | 8 | 0 | Edge address lifecycle |
| edgediscovery/allregions/\*\_test.go | ~15 | 0 | IPv4/IPv6 failover |
| cfapi/\*\_test.go | ~15 | 0 | JSON serialization contracts |
| credentials/\*\_test.go | ~9 | 0 | Origin cert encode/decode |
| token/\*\_test.go | ~10 | 0 | JWT handling, signal handler |
| features/selector\_test.go | 7 | 0 | Feature flag evaluation |
| config/\*\_test.go | ~3 | 0 | YAML/JSON config round-trip |
| metrics/\*\_test.go | 2 | 0 | Listener creation, readiness lifecycle |
| tracing/\*\_test.go | ~10 | 1 | Trace provider, identity parsing |
| validation/validation\_test.go | ~5 | 1 | URL/hostname normalization |
| ipaccess/access\_test.go | 5 | 0 | IP access rule matching |
| Other files | ~15 | 0 | Signal safety, file watch, backoff, overwatch, hello |
| **Total** | **~143** | **2** | |
