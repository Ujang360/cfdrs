# Wire Protocol — RPC and Datagram Formats

> Part of the [Wire Protocol Behavior Catalog](README.md).

## Cap'n Proto RPC Wire Contracts

### RPC Transport Bootstrap

RPC streams use `tunnelrpc.SafeTransport` wrapping an `io.ReadWriteCloser` stream. `SafeTransport` treats temporary read errors as recoverable to keep RPC sessions alive under transient faults. Client/server conn constructors (`NewClientConn`/`NewServerConn`) bootstrap Cap'n Proto RPC over the transport.

Primary evidence: [tunnelrpc/utils](../../../atoms/tunnelrpc/utils.md).

### Registration RPC Wire Format

| RPC method | Direction | Wire parameters | Wire response |
|---|---|---|---|
| `RegisterConnection` | cloudflared → edge | `TunnelAuth{AccountTag, TunnelSecret}`, `tunnelID uuid`, `connIndex uint8`, `ConnectionOptions{ClientInfo{ClientID, Features[], Version, Arch}, OriginLocalIP, ReplaceExisting, CompressionQuality, NumPreviousAttempts}` | `ConnectionDetails{UUID, Location, TunnelIsRemotelyManaged}` or `ConnectionError{cause, shouldRetry, retryAfter}` |
| `SendLocalConfiguration` | cloudflared → edge | `config []byte` (JSON) | *(void)* |
| `GracefulShutdown` | cloudflared → edge | `gracePeriod time.Duration` | *(void)* |

Schema: [tunnelrpc/proto/tunnelrpc.capnp](../../../atoms/tunnelrpc/proto/tunnelrpc.capnp.md).

Primary evidence: [tunnelrpc/registration_client](../../../atoms/tunnelrpc/registration_client.md), [tunnelrpc/registration_server](../../../atoms/tunnelrpc/registration_server.md), [tunnelrpc/pogs/registration_server](../../../atoms/tunnelrpc/pogs/registration_server.md).

### Session Management RPC Wire Format (v2)

| RPC method | Direction | Wire parameters |
|---|---|---|
| `RegisterUdpSession` | cloudflared → edge | `sessionID uuid`, `dstIP net.IP`, `dstPort uint16`, `closeIdleAfterHint time.Duration`, `traceContext string` |
| `UnregisterUdpSession` | cloudflared → edge | `sessionID uuid`, `message string` |
| `UpdateConfiguration` | edge → cloudflared | `version int32`, `config []byte` |

Two client types coexist: `CloudflaredClient` (full: session + config) and `SessionClient` (session only). Both accept `requestTimeout` and wrap `io.ReadWriteCloser`.

Primary evidence: [tunnelrpc/quic/cloudflared_client](../../../atoms/tunnelrpc/quic/cloudflared_client.md), [tunnelrpc/quic/session_client](../../../atoms/tunnelrpc/quic/session_client.md), [tunnelrpc/quic/session_server](../../../atoms/tunnelrpc/quic/session_server.md), [tunnelrpc/pogs/session_manager](../../../atoms/tunnelrpc/pogs/session_manager.md).

### RPC Error Wire Semantics

| Error pattern | Wire behavior |
|---|---|
| Retryable connection errors | `ConnectionError` with `shouldRetry=true` and optional `retryAfter` duration |
| Duplicate connection | Pre-mapped error detected by RPC adapter; triggers specific metric |
| Temporary transport reads | `SafeTransport` absorbs `net.Error.Temporary()` reads instead of tearing down the RPC conn |
| RPC metrics | Per-method handler observation + client latency timers via [tunnelrpc/metrics/metrics](../../../atoms/tunnelrpc/metrics/metrics.md) |

Primary evidence: [tunnelrpc/pogs/errors](../../../atoms/tunnelrpc/pogs/errors.md), [tunnelrpc/utils](../../../atoms/tunnelrpc/utils.md), [connection/control](../../../atoms/connection/control.md).

## Datagram v2 Wire Format

Datagram v2 uses a **suffix-muxed** scheme. The last byte of each datagram is a type discriminator, and session IDs are suffixed before the type byte.

### v2 Datagram Types

| Type byte (suffix) | Enum | Content |
|---|---|---|
| `0x00` | `DatagramTypeUDP` | Session payload: `[payload][16-byte sessionID][0x00]` |
| `0x01` | `DatagramTypeIP` | Raw IP packet: `[IP packet][0x01]` |
| `0x02` | `DatagramTypeIPWithTrace` | IP + tracing: `[IP packet][tracing identity][0x02]` |
| `0x03` | `DatagramTypeTracingSpan` | Tracing spans: `[protobuf spans][tracing identity][0x03]` |

### v2 Demux Flow

1. `ReceiveDatagram` from QUIC connection
2. Read last byte as `DatagramV2Type`
3. Strip type suffix
4. If `DatagramTypeUDP`: extract 16-byte session UUID suffix → dispatch to session channel
5. If `DatagramTypeIP`/`DatagramTypeIPWithTrace`/`DatagramTypeTracingSpan`: dispatch to packet channel

Session registration/unregistration is handled via Cap'n Proto RPC (SessionClient), not datagram frames.

Primary evidence: [quic/datagramv2](../../../atoms/quic/datagramv2.md), [connection/quic_datagram_v2](../../../atoms/connection/quic_datagram_v2.md), [datagramsession/session](../../../atoms/datagramsession/session.md).

## Datagram v3 Wire Format

Datagram v3 uses a **prefix-typed binary** scheme. The first byte is the datagram type, and all framing is big-endian encoded with explicit field layouts.

### v3 Datagram Types

| Type byte (prefix) | Enum | Meaning |
|---|---|---|
| `0x00` | `UDPSessionRegistrationType` | Session registration request |
| `0x01` | `UDPSessionPayloadType` | Session payload delivery |
| `0x02` | `ICMPType` | ICMP packet relay (v4 and v6) |
| `0x03` | `UDPSessionRegistrationResponseType` | Registration response |

### v3 Session Registration Frame

```text
 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |  Type (0x00)  |     Flags     |       Destination Port        |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |     Idle Duration Seconds     |                               |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               +
 |                                                               |
 +                     Session Identifier (16B)                  +
 |                                                               |
 +                                                               +
 |                                                               |
 +                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                               | Dest IPv4 (4B) or IPv6 (16B)  |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+- - - - - - - - - - - - - - - -+
 |                  (optional bundled payload)                    |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Flags byte:**

| Bit | Mask | Meaning |
|---|---|---|
| 0 | `0x01` | IPv6 (1) vs IPv4 (0) destination |
| 1 | `0x02` | Traced session |
| 2 | `0x04` | Bundled payload present |

Header sizes: IPv4 = 26 bytes, IPv6 = 38 bytes.

### v3 Session Payload Frame

```text
 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |  Type (0x01)  |                                               |
 +-+-+-+-+-+-+-+-+                                               +
 |                     Session Identifier (16B)                  |
 +                                               +-+-+-+-+-+-+-+-+
 |                                               |               |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               +
 .                         Payload                               .
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

Header size: 17 bytes (`datagramTypeLen` + 16-byte RequestID).

### v3 Registration Response Frame

```text
 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |  Type (0x03)  |   Resp Type   |                               |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               +
 |                     Session Identifier (16B)                  |
 +                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                               |         Error Length          |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 .                     Error Message (variable)                  .
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Response type codes:**

| Code | Enum | Meaning |
|---|---|---|
| `0x00` | `ResponseOk` | Session ready to proxy |
| `0x01` | `ResponseDestinationUnreachable` | Origin not reachable |
| `0x02` | `ResponseUnableToBindSocket` | Local UDP bind failed |
| `0x03` | `ResponseTooManyActiveFlows` | Flow limit exceeded |
| `0xFF` | `ResponseErrorWithMsg` | Generic error with message |

### v3 ICMP Frame

```text
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |  Type (0x02)  |       ICMP Payload ...        |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

ICMP datagrams carry raw ICMP packets (v4 or v6) prefixed by the single type byte.

### v3 Session Lifecycle (No RPC)

In datagram v3, session registration and response are embedded in datagram frames — no Cap'n Proto RPC is used. The v3 handler explicitly rejects `RegisterUdpSession`/`UnregisterUdpSession` RPC calls with `ErrUnsupportedRPCUDPRegistration` and `ErrUnsupportedRPCUDPUnregistration`.

Primary evidence: [quic/v3/datagram](../../../atoms/quic/v3/datagram.md), [quic/v3/muxer](../../../atoms/quic/v3/muxer.md), [quic/v3/session](../../../atoms/quic/v3/session.md), [quic/v3/icmp](../../../atoms/quic/v3/icmp.md), [connection/quic_datagram_v3](../../../atoms/connection/quic_datagram_v3.md).

## v2 vs v3 Datagram Protocol Comparison

| Aspect | Datagram v2 | Datagram v3 |
|---|---|---|
| Type encoding | Suffix (last byte) | Prefix (first byte) |
| Session ID encoding | 16-byte UUID suffix before type byte | 16-byte RequestID after type byte |
| Session registration | Cap'n Proto RPC (`SessionClient`) | Binary datagram frame (type `0x00`) with inline dest/flags |
| Session response | Cap'n Proto RPC return | Binary datagram frame (type `0x03`) with status code + error |
| ICMP support | Via `DatagramTypeIP` raw IP packets | Dedicated `ICMPType` (`0x02`) frame |
| Tracing support | `DatagramTypeIPWithTrace` / `DatagramTypeTracingSpan` with tracing identity suffix | Traced flag in registration, no dedicated span type |
| Payload bundling | Not supported | Optional bundled payload in registration frame |
| Session migration | Not supported | Supported (muxer detects registration for existing session) |
| Rate limiting | Not at datagram level | Muxer-level rate limiting on registrations |
| MTU default | `maxDatagramPayloadSize` (from QUIC frame size) | 1280 bytes (`maxDatagramPayloadLen`) |

Primary evidence: [quic/datagramv2](../../../atoms/quic/datagramv2.md), [quic/v3/datagram](../../../atoms/quic/v3/datagram.md), [quic/v3/muxer](../../../atoms/quic/v3/muxer.md).

## Management Wire Protocol

### HTTP Endpoints

| Path | Method | Wire behavior |
|---|---|---|
| `/ping` | GET | Returns 200 OK, no body |
| `/logs` | GET (WebSocket upgrade) | Streaming JSON log events with filter params |
| `/host_details` | GET | JSON system information |
| `/metrics` | GET | Prometheus exposition format |
| `/debug/pprof/*` | GET | Go pprof HTTP handlers |

### Management WebSocket Log Stream

Access is gated by JWT token validation (restricted to `*.cloudflare.com` CORS origins). Session limit = 1 concurrent. Idle timeout = 5 minutes. Heartbeat = 15 seconds.

Wire format over WebSocket: JSON-serialized log events matching the client's filter (level, event type, sampling rate).

Primary evidence: [management/service](../../../atoms/management/service.md), [management/events](../../../atoms/management/events.md), [management/session](../../../atoms/management/session.md), [management/middleware](../../../atoms/management/middleware.md), [management/token](../../../atoms/management/token.md).
