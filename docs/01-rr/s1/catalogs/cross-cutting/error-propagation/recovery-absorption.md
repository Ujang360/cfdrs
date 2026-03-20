# Error Propagation — Recovery and Absorption

> Part of the [Error Propagation Behavior Catalog](README.md).

## Panic Recovery Sites

cloudflared has exactly **two** structured panic recovery sites in the data path, plus one in the service layer:

### 1. `serveTunnel` Panic Recovery

Location: [supervisor/tunnel.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/supervisor/tunnel.go), `EdgeTunnelServer.serveTunnel()`.

```go
defer func() {
    if r := recover(); r != nil {
        err, ok = r.(error)
        if !ok {
            err = fmt.Errorf("ServeTunnel: %v", r)
        }
        err = errors.Wrapf(err, "stack trace: %s", string(debug.Stack()))
        recoverable = true
    }
}()
```

- **Behavior**: Any panic in the entire `serveTunnel` call tree is caught, wrapped with a stack trace, and returned as `recoverable = true`. The connection will be retried.
- **Quirk — Panics are always recoverable.** There is no panic that is treated as fatal at this level. Even a nil-pointer dereference deep in the proxy layer results in a retry.
- **Sentry integration**: Panics here are not sent to Sentry directly. Sentry reporting happens selectively via `reportErrorToSentry()` only for QUIC dial errors under specific FIPS + PQ conditions.

### 2. `stream.Pipe` / `PipeBidirectional` Panic Recovery

Location: [stream/stream.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/stream/stream.go), `unidirectionalStream()`.

```go
defer func() {
    if err := recover(); err != nil {
        if status.isAnyDone() {
            log.Debug().Msgf("recovered from panic in stream.Pipe for %s, ...")
        } else {
            log.Warn().Msgf("recovered from panic in stream.Pipe for %s, ...")
            sentry.CurrentHub().Recover(err)
            sentry.Flush(time.Second * 5)
        }
    }
}()
```

- **Behavior**: Panics in the bidirectional copy goroutines are caught. If one stream direction has already completed (`isAnyDone()`), the panic is treated as expected (debug log). If neither direction has completed, it is unexpected (warn log + Sentry report).
- **Quirk — This is the only site that reports to Sentry unconditionally on unexpected panics.** The `serveTunnel` recovery does _not_ report to Sentry; `reportErrorToSentry` is limited to QUIC dial errors.

### 3. `reportErrorToSentry` (Selective Sentry)

Location: [supervisor/tunnel.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/supervisor/tunnel.go), `EdgeTunnelServer.reportErrorToSentry()`.

This is _not_ a panic recovery but a selective error reporter with a very narrow filter:

```go
func (e *EdgeTunnelServer) reportErrorToSentry(err error, pqMode features.PostQuantumMode) {
    dialErr, ok := err.(*connection.EdgeQuicDialError)
    if ok {
        transportErr, ok := dialErr.Cause.(*quic.TransportError)
        if ok && transportErr.ErrorCode.IsCryptoError() &&
            fips.IsFipsEnabled() && pqMode == features.PostQuantumStrict {
            sentry.CaptureException(err)
        }
    }
}
```

Only fires when _all four conditions_ hold: `EdgeQuicDialError` → `quic.TransportError` → crypto error → FIPS enabled → PQ strict mode. This targets post-quantum handshake failures in FIPS environments.

### Panic Path Annotations (No Recovery)

Several sites have documented panic _paths_ without recovery:

| Location | Panic Trigger | Recovery | Evidence |
|---|---|---|---|
| [connection/header.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/connection/header.go) `mustInitRespMetaHeader()` | Malformed response metadata | None (caught by serveTunnel) | [atoms/connection/header](../../../atoms/connection/header.md) |
| [connection/http2.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/connection/http2.go) header state | Improper header state | None (caught by serveTunnel) | [atoms/connection/http2](../../../atoms/connection/http2.md) |
| [connection/header.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/connection/header.go) `DeserializeHeaders()` | Malformed input | None (caught by serveTunnel) | [atoms/connection/header](../../../atoms/connection/header.md) |

These all fall within the `serveTunnel` call tree and are caught by recovery site #1.

## Error Absorption Points

Error absorption occurs where errors are logged or handled locally but _not_ propagated to callers. These are decision points where Go's `if err != nil` leads to a log-and-continue rather than a return.

### Transport Layer Absorption

| Site | Absorbed Error | Mechanism | Evidence |
|---|---|---|---|
| `stream.unidirectionalStream` | Copy errors (`io.EOF`, connection close) | Logged at debug; `status.markUniStreamDone()` | [atoms/stream/stream](../../../atoms/stream/stream.md) |
| `datagramsession.Session.dstToTransport` | Non-close errors from `sendFunc` | Logged with variable severity; continue loop unless `closeSession = true` | [atoms/datagramsession/session](../../../atoms/datagramsession/session.md) |
| `datagramsession.Session.transportToDst` | Write errors to `dstConn` | Logged via `s.log.Err(err)` but still returns the error | [atoms/datagramsession/session](../../../atoms/datagramsession/session.md) |
| `quic/v3.session.writeLoop` | `os.ErrDeadlineExceeded` | Logged as warn; drops packet; continues loop (does not close session) | [atoms/quic/v3/session](../../../atoms/quic/v3/session.md) |
| `quic/v3.session.readLoop` | Connection closed errors (`net.ErrClosed`, `io.EOF`, `io.ErrUnexpectedEOF`) | Logged at debug; `closeSession(err)` to propagate via errChan | [atoms/quic/v3/session](../../../atoms/quic/v3/session.md) |
| `quic/v3.session.Write` | Channel full | Drops payload silently (metrics increment + error log); does not close session | [atoms/quic/v3/session](../../../atoms/quic/v3/session.md) |
| `connection.DialQuic` UDP socket | `udpConn.Close()` error | Silently ignored after successful QUIC dial wrapping | [atoms/connection/quic](../../../atoms/connection/quic.md) |

### Application Layer Absorption

| Site | Absorbed Error | Mechanism | Evidence |
|---|---|---|---|
| `proxy.applyIngressMiddleware` | Middleware filter results | When `result.ShouldFilterRequest`, writes status code, returns error as _handled_ (`return nil` at caller) | [atoms/proxy/proxy](../../../atoms/proxy/proxy.md) |
| `proxy.ProxyHTTP` (middleware filter path) | Filtered errors | `logRequestError` + `return nil` — caller sees success | [atoms/proxy/proxy](../../../atoms/proxy/proxy.md) |
| `logger.fallbackLogger` | Logger creation failure | Absorbed with fallback to default logger | [atoms/logger/create](../../../atoms/logger/create.md) |
| `management.IsClosed` | WebSocket close errors | Returns `true`/`false` classification; non-normal closures logged at debug | [atoms/management/events](../../../atoms/management/events.md) |
| `carrier.closeRespBody` | HTTP response body close error | Likely absorbed (close errors are rarely propagated) | [atoms/carrier/carrier](../../../atoms/carrier/carrier.md) |
| Observer `sendEvent` | Channel buffer full | Event dropped; warn log emitted | [atoms/connection/observer](../../../atoms/connection/observer.md) |
| `session.markActive` (both v2 and v3) | Channel full | Silently dropped via `select default` — acceptable precision loss | [atoms/datagramsession/session](../../../atoms/datagramsession/session.md), [atoms/quic/v3/session](../../../atoms/quic/v3/session.md) |

### Error Absorption Variance for Rust Port

| Go Pattern | Rust Concern |
|---|---|
| `log + continue` in loops | Must replicate: `tracing::warn!` + `continue` |
| `_ = expr` ignoring close errors | Explicit `let _ = ...;` required for clarity |
| `select { default: }` drop | Use `try_send().ok()` on bounded channels |
| `if err != nil { log; return nil }` (absorb + return success) | Careful: Rust's `Result` type makes absorption visible; must explicitly use `Ok(())` |
| Variable severity via `ErrVithVariableSeverity` interface | Implement as a trait: `fn log_level(&self) -> tracing::Level` |

## Error Wrapping Patterns

### `pkg/errors` Wrapping (40+ files)

The codebase uses `github.com/pkg/errors` (not `fmt.Errorf %w`) as the primary wrapping library. Key wrapping sites:

| Call Pattern | Purpose | Representative Files |
|---|---|---|
| `errors.Wrap(err, "context")` | Add context string to error chain | [proxy/proxy.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/proxy/proxy.go), [carrier/carrier.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/carrier/carrier.go), [stream/stream.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/stream/stream.go) |
| `errors.Wrapf(err, "format %s", arg)` | Add formatted context | `serveTunnel` panic recovery |
| `errors.New("msg")` | Create sentinel-like errors | Various |

- **Quirk — `pkg/errors` vs `fmt.Errorf %w`.** `pkg/errors.Wrap` captures a stack trace at the wrapping site. `fmt.Errorf %w` does not. The codebase predominantly uses `pkg/errors` for wrapping, which means Rust error chains should include `#[track_caller]` or `backtrace` support for parity.

### Datagram-Specific Wrapping

| Wrapper | Source | Purpose |
|---|---|---|
| `wrapMarshalErr(err)` | [quic/v3/datagram_errors.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/quic/v3/datagram_errors.go) | Adds "datagram marshal error:" prefix via `fmt.Errorf("%w")` |
| `wrapUnmarshalErr(err)` | [quic/v3/datagram_errors.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/quic/v3/datagram_errors.go) | Adds "datagram unmarshal error:" prefix via `fmt.Errorf("%w")` |

These use `fmt.Errorf %w` (not `pkg/errors`), making them the exception to the codebase pattern.

## Variable Severity Logging

The `ErrVithVariableSeverity` interface in `datagramsession/session.go` enables errors to declare their own log severity:

```go
type ErrVithVariableSeverity interface {
    error
    LogLevel() zerolog.Level
}
```

Used in the v2 datagram session's `dstToTransport` loop:

```go
level := zerolog.ErrorLevel
if variableErr, ok := err.(ErrVithVariableSeverity); ok {
    level = variableErr.LogLevel()
}
s.log.WithLevel(level).Err(err).Msg("Failed to send session payload...")
```

- **Quirk — The v3 datagram session does NOT use variable severity.** v3 (`quic/v3/session.go`) uses fixed log levels (`Debug`, `Warn`, `Error`) based on error classification functions like `isConnectionClosed(err)`.
- **Rust design note**: Implement as a trait `VariableSeverityError` with method `fn log_level(&self) -> tracing::Level`. Consider using `thiserror` with a custom trait bound.

## Connection-Closed Detection

Two independent implementations exist for detecting closed connections:

### v3 Implementation

```go
func isConnectionClosed(err error) bool {
    return errors.Is(err, net.ErrClosed) ||
           errors.Is(err, io.EOF) ||
           errors.Is(err, io.ErrUnexpectedEOF)
}
```

### v2 Implementation

The v2 session checks inline:

```go
if errors.Is(err, net.ErrClosed) || errors.Is(err, io.EOF) {
    s.log.Debug().Msg("Destination connection closed")
}
```

- **Quirk — v2 does not check `io.ErrUnexpectedEOF`.** This is a subtle behavioral difference between datagram v2 and v3.
