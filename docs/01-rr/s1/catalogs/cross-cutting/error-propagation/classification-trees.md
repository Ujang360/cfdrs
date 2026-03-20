# Error Propagation — Classification Trees

> Part of the [Error Propagation Behavior Catalog](README.md).

## Error Classification Decision Trees

### The `serveTunnel` Classifier

The central error classification site is `EdgeTunnelServer.serveTunnel()` in [supervisor/tunnel.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/supervisor/tunnel.go). It returns `(err error, recoverable bool)` — the two-value signature that drives retry/fallback decisions upstream.

```mermaid
flowchart TD
    ENTRY["serveTunnel returns (err, recoverable)"]
    NIL{"err == nil?"}
    SRTE{"ServerRegisterTunnelError?"}
    EQDIAL{"*EdgeQuicDialError?"}
    RECONN{"ReconnectSignal?"}
    CANCEL{"context.Canceled?"}
    UNREC{"unrecoverableError?"}
    DEFAULT["default error"]

    ENTRY --> NIL
    NIL -->|yes| RET_NIL["return nil, false"]
    NIL -->|no| SRTE
    SRTE -->|yes| PERM{"err.Permanent?"}
    PERM -->|yes| RET_FATAL_SRTE["return err.Cause, false\n(don't send to Sentry)"]
    PERM -->|no| RET_RETRY_SRTE["return err.Cause, true"]
    SRTE -->|no| EQDIAL
    EQDIAL -->|yes| RET_FATAL_DIAL["return err, false"]
    EQDIAL -->|no| RECONN
    RECONN -->|yes| DELAY["err.DelayBeforeReconnect()"]
    DELAY --> RET_RECONN["return err, true"]
    RECONN -->|no| CANCEL
    CANCEL -->|yes| RET_CANCEL["return err, false\n(debug-level log)"]
    CANCEL -->|no| UNREC
    UNREC -->|yes| RET_UNREC["return err, false"]
    UNREC -->|no| DEFAULT
    DEFAULT --> LOG["ConnAwareLogger.Err()"]
    LOG --> RET_DEFAULT["return err, true"]
```

**Key observations:**

- **Quirk — `ServerRegisterTunnelError` strips its wrapper.** The returned error is `err.Cause`, not the `ServerRegisterTunnelError` itself. The Sentry comment states: "Don't send registration error return from server to Sentry. They are logged on server side."
- **Quirk — `EdgeQuicDialError` is fatal at `serveTunnel` level** but retriable at `startFirstTunnel` level. The same error type has different recoverability at different stack depths.
- **Quirk — default branch is recoverable.** Unknown/uncaught error types are treated as recoverable (`true`) unless they implement `unrecoverableError`. This biases toward retry.

### The `startFirstTunnel` Classifier

`Supervisor.startFirstTunnel()` in [supervisor/supervisor.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/supervisor/supervisor.go) runs a retry loop with a type-switch that further classifies errors already processed by `serveTunnel`:

```mermaid
flowchart TD
    LOOP["for: retry loop"]
    ERR["err from Serve()"]
    CTX{"ctx.Err() != nil?"}
    NIL2{"err == nil?"}
    BACKOFF{"GetMaxBackoffDuration\nhas retry budget?"}
    UNAUTH{"err contains\n'Unauthorized'?"}
    TYPE{"type switch"}
    NOADDR["ErrNoAddressesLeft"]
    ALWAYS_RETRY["DupConnRegisterTunnelError\n*quic.IdleTimeoutError\n*quic.ApplicationError\nDialError\n*EdgeQuicDialError\n*ControlStreamError\n*StreamListenerError\n*DatagramManagerError"]
    BAIL["default: uncaught\n⟶ return (bail startup)"]
    STATIC{"isStaticEdge?"}

    LOOP --> ERR
    ERR --> CTX
    CTX -->|yes| RETURN_CTX["return"]
    CTX -->|no| NIL2
    NIL2 -->|yes| RETURN_NIL["return"]
    NIL2 -->|no| BACKOFF
    BACKOFF -->|no budget| RETURN_BACK["return"]
    BACKOFF -->|has budget| UNAUTH
    UNAUTH -->|yes| LOOP
    UNAUTH -->|no| TYPE
    TYPE -->|ErrNoAddressesLeft| STATIC
    STATIC -->|static| LOOP
    STATIC -->|dynamic| RETURN_NOADDR["return"]
    TYPE -->|always-retry types| LOOP
    TYPE -->|default| BAIL
```

**Key observations:**

- **Quirk — Unauthorized is string-matched.** `strings.Contains(err.Error(), "Unauthorized")` treats any error whose message contains "Unauthorized" as transient. This is a heuristic for edge propagation lag on new tunnels.
- **Quirk — `ErrNoAddressesLeft` depends on edge mode.** Static edge configurations retry forever; dynamic configurations bail immediately. Same error, different outcomes.
- **Quirk — 8 error types are always-retried.** These include QUIC-specific errors (`*quic.IdleTimeoutError`, `*quic.ApplicationError`) and all the `connection` package error types. Everything else bails from startup.

### The `ipAddrFallback` Classifier

`ipAddrFallback.ShouldGetNewAddress()` maps errors to address-rotation decisions:

```mermaid
flowchart TD
    ERR["err from serveTunnel"]
    NIL3{"nil?"}
    DUP{"DupConnRegisterTunnelError\nor *quic.IdleTimeoutError?"}
    DIAL{"DialError\nor *EdgeQuicDialError?"}
    DEFAULT2["default"]

    ERR --> NIL3
    NIL3 -->|yes| KEEP["keep current IP"]
    NIL3 -->|no| DUP
    DUP -->|yes| ROTATE_CLEAN["rotate IP, no connectivity error"]
    DUP -->|no| DIAL
    DIAL -->|yes| RETRY_CHECK{"retries >= maxRetries?"}
    RETRY_CHECK -->|yes| ROTATE_MAX["rotate IP + ConnectivityError(maxReached=true)\n⟶ forces protocol fallback"]
    RETRY_CHECK -->|no| ROTATE_INC["rotate IP + ConnectivityError(maxReached=false)\nincrement retry counter"]
    DIAL -->|no| DEFAULT2
    DEFAULT2 --> KEEP
```

**Key observations:**

- **Quirk — `DupConnRegisterTunnelError` and `*quic.IdleTimeoutError` rotate silently.** No `ConnectivityError` is produced — the retry counter is not affected. These are treated as "expected" address issues.
- **Quirk — Only `DialError` and `*EdgeQuicDialError` count toward retry exhaustion.** All other error types keep the current IP address unchanged. This creates a narrow funnel where only network-level dial failures escalate to protocol fallback.

### The `isQuicBroken` Classifier

Determines whether QUIC should be abandoned for HTTP/2 fallback:

| Error pattern | Matches | Mechanism |
|---|---|---|
| `*quic.IdleTimeoutError` | Yes | `errors.As` unwrap |
| `*quic.TransportError` | Yes | `errors.As` unwrap |
| `*net.OpError` wrapping "operation not permitted" | Yes | `errors.As` + `strings.Contains` on inner error |
| Everything else | No | Default: QUIC is not broken |

- **Quirk — "operation not permitted" is string-matched inside `*net.OpError`.** A low-level kernel error string comparison drives a protocol-level decision. This catches cases where egress UDP to port 7844 is blocked by firewalls.

## Error Propagation Through the Connection Lifecycle

### Dual-Return Pattern: `(error, bool)`

The core propagation mechanism is the `(error, recoverable bool)` return pair used by:

- `serveTunnel(ctx, ...) (err error, recoverable bool)`
- `serveConnection(ctx, ...) (err error, recoverable bool)`
- `serveQUIC(ctx, ...) (err error, recoverable bool)`

This pattern is _not_ a standard Go idiom — it is cloudflared-specific. The `recoverable` flag drives a critical decision point in `EdgeTunnelServer.Serve()`:

1. `recoverable = false` → return error to supervisor → supervisor decides (typically bail for non-first connections)
2. `recoverable = true` → enter backoff wait → check if protocol fallback is needed → retry connection

### The `shouldFallbackProtocol` Decision Chain

After `serveTunnel` returns, `Serve()` computes `shouldFallbackProtocol`:

1. `ipAddrFallback.ShouldGetNewAddress(connIndex, err)` produces `(shouldRotateEdgeIP, cErr)`
2. If `cErr` is a `*ConnectivityError` with `HasReachedMaxRetries() == true`, force protocol fallback
3. Wait for backoff timer
4. If `shouldFallbackProtocol == true` AND no connection has succeeded with the current protocol → call `selectNextProtocol()`
5. `selectNextProtocol()` checks `isQuicBroken(cause)` and `selector.Fallback()` availability

### `errgroup.Wait()` Error Aggregation

HTTP/2 and QUIC connections use `golang.org/x/sync/errgroup`:

```go
errGroup.Go(func() error { return h2conn.Serve(serveCtx) })
errGroup.Go(func() error { return listenReconnect(serveCtx, ...) })
return errGroup.Wait()
```

- `errgroup.Wait()` returns the **first non-nil error** from any goroutine and cancels the derived context.
- **Quirk — `listenReconnect` never returns a real error in production.** It returns `nil` on graceful shutdown and `ReconnectSignal` on forced reconnect. The `ReconnectSignal` error is what causes the errgroup to cancel `h2conn.Serve()`.

### Connection Close Error Chain

When QUIC connections close, a reference-counted close pattern is used:

```go
type wrapCloseableConnQuicConnection struct {
    quic.Connection
    udpConn *net.UDPConn
}

func (w *wrapCloseableConnQuicConnection) CloseWithError(errorCode, reason) error {
    err := w.Connection.CloseWithError(errorCode, reason)
    w.udpConn.Close()  // UDP close error silently ignored
    return err
}
```

- **Quirk — UDP close errors are always silently discarded.** The QUIC-level close error is propagated; the underlying UDP socket close error is not.
