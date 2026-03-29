# Concurrency — Goroutine Spawn Site Inventory

> Part of the [Concurrency Behavior Catalog](README.md).

## Process-Scoped Goroutines

| Spawn site | Spawned goroutine | Lifetime | Termination | Evidence |
| --- | --- | --- | --- | --- |
| `cmd tunnel` main | `Supervisor.Run()` | process | ctx cancel or all tunnels exit | [atoms/cmd/cloudflared/tunnel/cmd](../../../atoms/cmd/cloudflared/tunnel/cmd.md) |
| `cmd tunnel` main | signal handler via `waitForSignal()` | process | OS signal received | [atoms/cmd/cloudflared/tunnel/signal](../../../atoms/cmd/cloudflared/tunnel/signal.md) |
| `cmd tunnel` main | `stdinControl()` stdin reader | process | EOF or line read | [atoms/cmd/cloudflared/tunnel/cmd](../../../atoms/cmd/cloudflared/tunnel/cmd.md) |
| `cmd tunnel` main | `notifySystemd()` | process | ctx cancel | [atoms/cmd/cloudflared/tunnel/cmd](../../../atoms/cmd/cloudflared/tunnel/cmd.md) |
| `NewAppManager` | `go serviceRun(service)` per service | service-scoped | `Service.Run()` returns | [atoms/overwatch/app_manager](../../../atoms/overwatch/app_manager.md) |
| `icmpRouter.Serve()` | `go ipv4Proxy.Serve(ctx)` | process | ctx cancel | [atoms/ingress/origin_icmp_proxy](../../../atoms/ingress/origin_icmp_proxy.md) |
| `icmpRouter.Serve()` | `go ipv6Proxy.Serve(ctx)` | process | ctx cancel | [atoms/ingress/origin_icmp_proxy](../../../atoms/ingress/origin_icmp_proxy.md) |

## Tunnel-Scoped Goroutines (Supervisor Layer)

| Spawn site | Spawned goroutine | Lifetime | Termination | Evidence |
| --- | --- | --- | --- | --- |
| `Supervisor.Run()` | `go s.startFirstTunnel(ctx, connectedSignal)` | tunnel | first connection success or fatal error | [atoms/supervisor/supervisor](../../../atoms/supervisor/supervisor.md) |
| `Supervisor.Run()` backoff timer | `go s.startTunnel(ctx, i, signal)` (×N-1) | tunnel | `EdgeTunnelServer.Serve()` returns | [atoms/supervisor/supervisor](../../../atoms/supervisor/supervisor.md) |
| `EdgeTunnelServer.Serve()` | `go connectedFuse.Await()` notifier | connection | fuse fires or defer fires false | [atoms/supervisor/tunnel](../../../atoms/supervisor/tunnel.md) |

## Connection-Scoped Goroutines (QUIC Mode)

| Spawn site | Spawned goroutine | Lifetime | Termination | Evidence |
| --- | --- | --- | --- | --- |
| `serveQUIC()` errgroup | control stream handler | connection | unregistration or ctx cancel | [atoms/supervisor/tunnel](../../../atoms/supervisor/tunnel.md) |
| `serveQUIC()` errgroup | `tunnelConn.Serve(serveCtx)` | connection | ctx cancel, `ControlStreamError` | [atoms/connection/quic_connection](../../../atoms/connection/quic_connection.md) |
| `serveQUIC()` errgroup | `listenReconnect()` | connection | reconnect signal, graceful shutdown, or ctx cancel | [atoms/supervisor/tunnel](../../../atoms/supervisor/tunnel.md) |
| `quicConnection.Serve()` errgroup | control stream via `serveControlStream()` | connection | unregistration | [atoms/connection/quic_connection](../../../atoms/connection/quic_connection.md) |
| `quicConnection.Serve()` errgroup | `acceptStream(ctx)` loop | connection | ctx cancel or accept error | [atoms/connection/quic_connection](../../../atoms/connection/quic_connection.md) |
| `quicConnection.Serve()` errgroup | datagram handler `.Serve(ctx)` | connection | ctx cancel | [atoms/connection/quic_connection](../../../atoms/connection/quic_connection.md) |
| `acceptStream()` loop | `go q.runStream(quicStream)` per stream | request | stream EOF or error | [atoms/connection/quic_connection](../../../atoms/connection/quic_connection.md) |

## Connection-Scoped Goroutines (HTTP/2 Mode)

| Spawn site | Spawned goroutine | Lifetime | Termination | Evidence |
| --- | --- | --- | --- | --- |
| `serveHTTP2()` errgroup | `h2conn.Serve(serveCtx)` | connection | server close or ctx cancel | [atoms/supervisor/tunnel](../../../atoms/supervisor/tunnel.md) |
| `serveHTTP2()` errgroup | `listenReconnect()` | connection | reconnect signal, graceful shutdown, or ctx cancel | [atoms/supervisor/tunnel](../../../atoms/supervisor/tunnel.md) |
| `HTTP2Connection.Serve()` | per-request `ServeHTTP()` (implicit via `http2.Server`) | request | handler returns | [atoms/connection/http2](../../../atoms/connection/http2.md) |

## Datagram v3 Goroutines (Per QUIC Connection)

| Spawn site | Spawned goroutine | Lifetime | Termination | Evidence |
| --- | --- | --- | --- | --- |
| `datagramConn.Serve()` | `go c.pollDatagrams(readCtx)` | connection | readCtx cancel or read error | [atoms/quic/v3/muxer](../../../atoms/quic/v3/muxer.md) |
| `datagramConn.Serve()` | `go c.processICMPDatagrams(readCtx)` | connection | readCtx cancel | [atoms/quic/v3/muxer](../../../atoms/quic/v3/muxer.md) |
| `datagramConn.Serve()` main loop | `go c.handleSessionRegistrationDatagram(connCtx, reg, &logger)` per session | session | `session.Serve()` returns | [atoms/quic/v3/muxer](../../../atoms/quic/v3/muxer.md) |
| `session.Serve()` | `go s.writeLoop()` | session | `closeWrite` channel closed | [atoms/quic/v3/session](../../../atoms/quic/v3/session.md) |
| `session.Serve()` | `go s.readLoop()` | session | origin socket closed | [atoms/quic/v3/session](../../../atoms/quic/v3/session.md) |

## Datagram v2 Goroutines (Per QUIC Connection)

| Spawn site | Spawned goroutine | Lifetime | Termination | Evidence |
| --- | --- | --- | --- | --- |
| `datagramV2Connection.Serve()` errgroup | `sessionManager.Serve(ctx)` | connection | ctx cancel | [atoms/connection/quic_datagram_v2](../../../atoms/connection/quic_datagram_v2.md) |
| `datagramV2Connection.Serve()` errgroup | `datagramMuxer.ServeReceive(ctx)` | connection | ctx cancel | [atoms/connection/quic_datagram_v2](../../../atoms/connection/quic_datagram_v2.md) |
| `datagramV2Connection.Serve()` errgroup | `packetRouter.Serve(ctx)` | connection | ctx cancel | [atoms/connection/quic_datagram_v2](../../../atoms/connection/quic_datagram_v2.md) |
| `RegisterUdpSession()` | `go serveUDPSession(session, ...)` per session | session | session idle, close, or error | [atoms/connection/quic_datagram_v2](../../../atoms/connection/quic_datagram_v2.md) |
| `Session.Serve()` (v2) | `go dstToTransport()` reader | session | EOF or `dstConn.Close()` | [atoms/datagramsession/session](../../../atoms/datagramsession/session.md) |

## Stream Piping Goroutines (Per Proxied Request)

| Spawn site | Spawned goroutine | Lifetime | Termination | Evidence |
| --- | --- | --- | --- | --- |
| `PipeBidirectional()` | `go unidirectionalStream(downstream, upstream, ...)` | request | EOF, error, or peer close | [atoms/stream/stream](../../../atoms/stream/stream.md) |
| `PipeBidirectional()` | `go unidirectionalStream(upstream, downstream, ...)` | request | EOF, error, or peer close | [atoms/stream/stream](../../../atoms/stream/stream.md) |

## Management WebSocket Goroutines (Per Client)

| Spawn site | Spawned goroutine | Lifetime | Termination | Evidence |
| --- | --- | --- | --- | --- |
| `logs()` handler | `go m.readEvents(c, ctx, events)` | websocket-scoped | read error or ctx cancel | [atoms/management/service](../../../atoms/management/service.md) |
| `logs()` handler | `go m.streamLogs(c, ctx, session)` | streaming-scoped | session.Stop() or ctx cancel | [atoms/management/service](../../../atoms/management/service.md) |
| `logs()` select loop | `go c.Ping(ctx)` | fire-and-forget | ping completes | [atoms/management/service](../../../atoms/management/service.md) |
