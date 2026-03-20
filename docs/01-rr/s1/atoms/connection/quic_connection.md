# Behavior Atom: connection/quic_connection.go

## Source Anchor

- Go source: [cloudflare/cloudflared@2026.3.0/connection/quic_connection.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/connection/quic_connection.go)
- Package: connection
- Module group: connection

## Behavioral Responsibility

Transport/protocol behavior for edge-origin data and control flows.

## Entry Points

- NewTunnelConnection(ctx context.Context, conn quic.Connection, connIndex uint8, orchestrator Orchestrator, datagramSessionHandler DatagramSessionHandler, controlStreamHandler ControlStreamHandler, connOptions *client.ConnectionOptionsSnapshot, rpcTimeout time.Duration, streamWriteTimeout time.Duration, gracePeriod time.Duration, logger*zerolog.Logger) TunnelConnection (line 58)
- (*quicConnection) Serve(ctx context.Context) error (line 87)
- (*quicConnection) Close() (line 151)
- (*quicConnection) UpdateConfiguration(ctx context.Context, version int32, config []byte)*pogs.UpdateConfigurationResponse (line 250)
- (*streamReadWriteAcker) AckConnection(tracePropagation string) error (line 262)
- (*httpResponseAdapter) AddTrailer(trailerName string, trailerValue string) (line 286)
- (*httpResponseAdapter) WriteRespHeaders(status int, header http.Header) error (line 290)
- (*httpResponseAdapter) Write(p []byte) (int, error) (line 303)
- (*httpResponseAdapter) Header() http.Header (line 311)
- (*httpResponseAdapter) Flush() (line 316)
- (*httpResponseAdapter) WriteHeader(status int) (line 318)
- (*httpResponseAdapter) Hijack() (net.Conn,*bufio.ReadWriter, error) (line 322)
- (*httpResponseAdapter) WriteErrorResponse(err error) (line 331)
- (*httpResponseAdapter) WriteConnectResponseData(respErr error, metadata ...pogs.Metadata) error (line 335)
- (*nopCloserReadWriter) Read(p []byte) (n int, err error) (line 421)
- (*nopCloserReadWriter) Close() error (line 438)

## Internal Function Surface

- (*quicConnection) serveControlStream(ctx context.Context, controlStream quic.Stream) error (line 146)
- (*quicConnection) acceptStream(ctx context.Context) error (line 155)
- (*quicConnection) runStream(quicStream quic.Stream) (line 169)
- (*quicConnection) handleDataStream(ctx context.Context, stream*rpcquic.RequestServerStream) error (line 189)
- (*quicConnection) dispatchRequest(ctx context.Context, stream*rpcquic.RequestServerStream, request *pogs.ConnectRequest) (err error, connectResponseSent bool) (line 220)
- newHTTPResponseAdapter(s *rpcquic.RequestServerStream) httpResponseAdapter (line 282)
- buildHTTPRequest(ctx context.Context, connectRequest *pogs.ConnectRequest, body io.ReadCloser, connIndex uint8, log*zerolog.Logger) (*tracing.TracedHTTPRequest, error) (line 340)
- setContentLength(req *http.Request) error (line 392)
- isTransferEncodingChunked(req *http.Request) bool (line 400)

## Input Contract

- HTTP requests
- func-param:body io.ReadCloser
- func-param:config []byte
- func-param:conn quic.Connection
- func-param:connIndex uint8
- func-param:connOptions *client.ConnectionOptionsSnapshot
- func-param:connectRequest *pogs.ConnectRequest
- func-param:controlStream quic.Stream
- func-param:controlStreamHandler ControlStreamHandler
- func-param:ctx context.Context
- func-param:datagramSessionHandler DatagramSessionHandler
- func-param:err error
- func-param:gracePeriod time.Duration
- func-param:header http.Header
- func-param:log *zerolog.Logger
- func-param:logger *zerolog.Logger
- func-param:metadata ...pogs.Metadata
- func-param:orchestrator Orchestrator
- func-param:p []byte
- func-param:quicStream quic.Stream
- func-param:req *http.Request
- func-param:request *pogs.ConnectRequest
- func-param:respErr error
- func-param:rpcTimeout time.Duration
- func-param:s *rpcquic.RequestServerStream
- func-param:status int
- func-param:stream *rpcquic.RequestServerStream
- func-param:streamWriteTimeout time.Duration
- func-param:tracePropagation string
- func-param:trailerName string
- func-param:trailerValue string
- func-param:version int32

## Output Contract

- HTTP response writes
- return:*bufio.ReadWriter
- return:*pogs.UpdateConfigurationResponse
- return:*tracing.TracedHTTPRequest
- return:TunnelConnection
- return:bool
- return:connectResponseSent bool
- return:err error
- return:error
- return:http.Header
- return:httpResponseAdapter
- return:int
- return:n int
- return:net.Conn
- stdout/stderr or structured logs

## Side Effects and State Transitions

- network I/O
- concurrency primitives
- timers and scheduling

## Branching and Failure Semantics

- Branch density: if=27, switch=1, select=1
- error-return paths
- fallback/default branches

## Import and Dependency Surface

- bufio
- context
- errors
- fmt
- github.com/cloudflare/cloudflared/client
- github.com/cloudflare/cloudflared/flow
- github.com/cloudflare/cloudflared/quic
- github.com/cloudflare/cloudflared/tracing
- github.com/cloudflare/cloudflared/tunnelrpc/pogs
- github.com/cloudflare/cloudflared/tunnelrpc/quic
- github.com/quic-go/quic-go
- github.com/rs/zerolog
- golang.org/x/sync/errgroup
- io
- net
- net/http
- strconv
- strings
- sync/atomic
- time

## Go-Impl Flow (Intra-file)

```mermaid
flowchart TD
    F1["NewTunnelConnection"]
    F2["*quicConnection.Serve"]
    F3["*quicConnection.serveControlStream"]
    F4["*quicConnection.Close"]
    F5["*quicConnection.acceptStream"]
    F6["*quicConnection.runStream"]
    F7["*quicConnection.handleDataStream"]
    F8["*quicConnection.dispatchRequest"]
    F9["*quicConnection.UpdateConfiguration"]
    F10["*streamReadWriteAcker.AckConnection"]
    F11["newHTTPResponseAdapter"]
    F12["*httpResponseAdapter.AddTrailer"]
    F13["*httpResponseAdapter.WriteRespHeaders"]
    F14["*httpResponseAdapter.Write"]
    F15["*httpResponseAdapter.Header"]
    F16["*httpResponseAdapter.Flush"]
    F17["*httpResponseAdapter.WriteHeader"]
    F18["*httpResponseAdapter.Hijack"]
    F19["*httpResponseAdapter.WriteErrorResponse"]
    F20["*httpResponseAdapter.WriteConnectResponseData"]
    F21["buildHTTPRequest"]
    F22["setContentLength"]
    F23["isTransferEncodingChunked"]
    F24["*nopCloserReadWriter.Read"]
    F25["*nopCloserReadWriter.Close"]
    F8 --> F21
    F8 --> F11
    F21 --> F22
    F21 --> F23
```

## Accuracy Notes

- Generated from Go AST parsing and source text pattern extraction.
- Source link is authoritative for disputed semantics; keep this atom synchronized with the linked file.

## Rust Porting Notes

- **QUIC connection**: `quic-go.Connection` → `quinn::Connection` with `accept_bi()`/`accept_uni()` for stream acceptance.
- **Task group**: `errgroup.Group` for concurrent stream+datagram serving → `tokio::task::JoinSet` with first-error propagation.
- **HTTP response adapter**: `httpResponseAdapter` implementing `http.ResponseWriter` → custom struct wrapping `quinn::SendStream` with typed header/status writing.
- **Stream ack**: `streamReadWriteAcker.AckConnection` writes trace propagation over QUIC stream → implement as async `write_all` on the `SendStream`.
- **Atomic config version**: `sync/atomic` for config update tracking → `std::sync::atomic::AtomicI32`.
- **Timeout handling**: `rpcTimeout` and `streamWriteTimeout` → `tokio::time::timeout` wrapping each RPC call or stream write.
- **Quirk — nopCloserReadWriter**: Adapter that wraps a reader to add a no-op `Close()` → unnecessary in Rust if the type doesn't require `AsyncWrite + Close`; otherwise use a newtype with no-op `Drop`.
- **Quirk — 27 if-branches**: Dense error handling in stream acceptance — model as `match` on typed `QuicStreamError` enum for clarity.
