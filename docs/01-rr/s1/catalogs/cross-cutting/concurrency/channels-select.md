# Concurrency — Channels and Select Loops

> Part of the [Concurrency Behavior Catalog](README.md).

## Channel Inventory

### Shutdown and Signaling Channels

| Channel | Type | Buffer | Producers | Consumers | Mechanism | Evidence |
| --- | --- | --- | --- | --- | --- | --- |
| `graceShutdownC` | `chan struct{}` | 0 | OS signal handler | all `Serve` loops, `listenReconnect` | close-broadcast | [atoms/cmd/cloudflared/tunnel/signal](../../../atoms/cmd/cloudflared/tunnel/signal.md) |
| `reconnectCh` | `chan ReconnectSignal` | 0 | external control, management | `listenReconnect` per connection | value send | [atoms/supervisor/external_control](../../../atoms/supervisor/external_control.md) |
| `tunnelErrors` | `chan tunnelError` | 0 | `startFirstTunnel`, `startTunnel` | `Supervisor.Run` select loop | value send | [atoms/supervisor/supervisor](../../../atoms/supervisor/supervisor.md) |
| `nextConnectedSignal` | `chan struct{}` | 0 | `signal.Signal.Notify()` | `Supervisor.Run` select loop | close-broadcast | [atoms/supervisor/supervisor](../../../atoms/supervisor/supervisor.md) |
| `signal.Signal.ch` | `chan struct{}` | 0 | `Signal.Notify()` (via `sync.Once`) | `Signal.Wait()` callers | close-broadcast (one-shot) | [atoms/signal/safe_signal](../../../atoms/signal/safe_signal.md) |
| `booleanFuse` internal | `chan struct{}` | 0 | `Fuse(bool)` | `Await()` goroutine | close-broadcast (one-shot) | [atoms/supervisor/fuse](../../../atoms/supervisor/fuse.md) |

### Data Flow Channels

| Channel | Type | Buffer | Producers | Consumers | Evidence |
| --- | --- | --- | --- | --- | --- |
| `datagrams` (v3) | `chan []byte` | 16 | `pollDatagrams` | `datagramConn.Serve` main loop | [atoms/quic/v3/muxer](../../../atoms/quic/v3/muxer.md) |
| `icmpDatagramChan` (v3) | `chan *ICMPDatagram` | 128 | `handleICMPPacket` | `processICMPDatagrams` | [atoms/quic/v3/muxer](../../../atoms/quic/v3/muxer.md) |
| `readErrors` (v3) | `chan error` | 2 | `pollDatagrams` | `datagramConn.Serve` main loop | [atoms/quic/v3/muxer](../../../atoms/quic/v3/muxer.md) |
| `writeChan` (v3 session) | `chan []byte` | 512 | `session.Write()` (edge payloads) | `writeLoop` | [atoms/quic/v3/session](../../../atoms/quic/v3/session.md) |
| `closeWrite` (v3 session) | `chan struct{}` | 0 | `closeFn` | `writeLoop` | [atoms/quic/v3/session](../../../atoms/quic/v3/session.md) |
| `errChan` (v3 session) | `chan error` | 3 | read loop, write loop, `closeFn` | `waitForCloseCondition` | [atoms/quic/v3/session](../../../atoms/quic/v3/session.md) |
| `activeAtChan` (v3 session) | `chan time.Time` | 1 | `markActive()` (non-blocking) | `waitForCloseCondition` | [atoms/quic/v3/session](../../../atoms/quic/v3/session.md) |
| `contextChan` (v3 session) | `chan context.Context` | 0 | `Migrate()` | `waitForCloseCondition` | [atoms/quic/v3/session](../../../atoms/quic/v3/session.md) |
| `sessionDemuxChan` (v2) | `chan *packet.Session` | 16 | `datagramMuxer.ServeReceive` | `sessionManager.Serve` | [atoms/connection/quic_datagram_v2](../../../atoms/connection/quic_datagram_v2.md) |
| `registrationChan` (v2) | `chan *registerSessionEvent` | 0 | `RegisterSession` callers | `manager.Serve` | [atoms/datagramsession/manager](../../../atoms/datagramsession/manager.md) |
| `unregistrationChan` (v2) | `chan *unregisterSessionEvent` | 0 | `UnregisterSession` callers | `manager.Serve` | [atoms/datagramsession/manager](../../../atoms/datagramsession/manager.md) |
| `closedChan` (v2 manager) | `chan struct{}` | 0 | manager shutdown | `RegisterSession`, `UnregisterSession` callers | [atoms/datagramsession/manager](../../../atoms/datagramsession/manager.md) |
| `activeAtChan` (v2 session) | `chan time.Time` | 2 | `markActive()` (non-blocking) | `waitForCloseCondition` | [atoms/datagramsession/session](../../../atoms/datagramsession/session.md) |
| `closeChan` (v2 session) | `chan error` | 2 | `dstToTransport`, `close()` | `waitForCloseCondition` | [atoms/datagramsession/session](../../../atoms/datagramsession/session.md) |
| `events` (management) | `chan *ClientEvent` | 0 | `readEvents` goroutine | `logs()` handler select loop | [atoms/management/service](../../../atoms/management/service.md) |
| `errC` (cmd) | `chan error` | 0 | server startup goroutine | `waitToShutdown` | [atoms/cmd/cloudflared/tunnel/cmd](../../../atoms/cmd/cloudflared/tunnel/cmd.md) |

### System Service Channels (Windows)

| Channel | Type | Buffer | Producers | Consumers | Evidence |
| --- | --- | --- | --- | --- | --- |
| `r` (svc requests) | `chan svc.ChangeRequest` | OS-provided | Windows SCM | `Execute()` loop | [atoms/cmd/cloudflared/windows_service](../../../atoms/cmd/cloudflared/windows_service.md) |
| `statusChan` | `chan svc.Status` | OS-provided | `Execute()` | Windows SCM | [atoms/cmd/cloudflared/windows_service](../../../atoms/cmd/cloudflared/windows_service.md) |

## Select Loop Inventory

### Supervisor Select Loops

| Location | Branches | Purpose | Evidence |
| --- | --- | --- | --- |
| `Supervisor.Run` main loop | `ctx.Done`, `tunnelErrors`, `backoffTimer`, `nextConnectedSignal` | Coordinate HA tunnel lifecycle: error handling, backoff retry, connection success | [atoms/supervisor/supervisor](../../../atoms/supervisor/supervisor.md) |
| `listenReconnect()` | `reconnectCh`, `gracefulShutdownCh`, `ctx.Done` | Wait for reconnect signal or shutdown | [atoms/supervisor/tunnel](../../../atoms/supervisor/tunnel.md) |
| `EdgeTunnelServer.Serve` retry | `ctx.Done`, `gracefulShutdownC`, `protocolFallback.BackoffTimer()` | Backoff wait between retry attempts | [atoms/supervisor/tunnel](../../../atoms/supervisor/tunnel.md) |

### Datagram v3 Select Loops

| Location | Branches | Purpose | Evidence |
| --- | --- | --- | --- |
| `datagramConn.Serve` main loop | `ctx.Done`, `connCtx.Done`, `readErrors`, `datagrams` | Demux incoming datagrams by type | [atoms/quic/v3/muxer](../../../atoms/quic/v3/muxer.md) |
| `processICMPDatagrams` | `ctx.Done`, `icmpDatagramChan` | Write ICMP packets to origin | [atoms/quic/v3/muxer](../../../atoms/quic/v3/muxer.md) |
| `session.waitForCloseCondition` | `connCtx.Done`, `contextChan`, `errChan`, `checkIdleTimer.C`, `activeAtChan` | Idle timeout, migration, error, close | [atoms/quic/v3/session](../../../atoms/quic/v3/session.md) |
| `session.writeLoop` | `closeWrite`, `writeChan` | Write payloads to origin or stop | [atoms/quic/v3/session](../../../atoms/quic/v3/session.md) |

### Datagram v2 Select Loops

| Location | Branches | Purpose | Evidence |
| --- | --- | --- | --- |
| `manager.Serve` | `ctx.Done`, `receiveChan`, `registrationChan`, `unregistrationChan` | Session lifecycle management | [atoms/datagramsession/manager](../../../atoms/datagramsession/manager.md) |
| `Session.waitForCloseCondition` | `ctx.Done`, `closeChan`, `checkIdleTicker.C`, `activeAtChan` | Idle timeout and close | [atoms/datagramsession/session](../../../atoms/datagramsession/session.md) |
| `manager.RegisterSession` | `ctx.Done`, `registrationChan`, `closedChan` | Timeout-guarded registration | [atoms/datagramsession/manager](../../../atoms/datagramsession/manager.md) |
| `manager.UnregisterSession` | `ctx.Done`, `unregistrationChan`, `closedChan` | Timeout-guarded unregistration | [atoms/datagramsession/manager](../../../atoms/datagramsession/manager.md) |

### Management Select Loop

| Location | Branches | Purpose | Evidence |
| --- | --- | --- | --- |
| `logs()` handler | `ctx.Done`, `events`, `StartStreaming` / `StopStreaming`, `ping.C`, `idle.C` | WebSocket session orchestration with idle timeout and keepalive | [atoms/management/service](../../../atoms/management/service.md) |

### Command-Level Select Loops

| Location | Branches | Purpose | Evidence |
| --- | --- | --- | --- |
| `Supervisor.Run` initial wait | `ctx.Done`, `tunnelErrors`, `gracefulShutdownC`, `connectedSignal.Wait()` | Wait for first tunnel to connect before spawning HA goroutines | [atoms/supervisor/supervisor](../../../atoms/supervisor/supervisor.md) |
| `waitToShutdown()` | `errC`, `graceShutdownC`, signal handler | Top-level exit coordination | [atoms/cmd/cloudflared/tunnel/cmd](../../../atoms/cmd/cloudflared/tunnel/cmd.md) |
| `windowsService.Execute` | `r` (service requests), internal state | Windows service control | [atoms/cmd/cloudflared/windows_service](../../../atoms/cmd/cloudflared/windows_service.md) |
