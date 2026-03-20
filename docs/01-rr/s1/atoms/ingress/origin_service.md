# Behavior Atom: ingress/origin_service.go

## Source Anchor

- Go source: [cloudflare/cloudflared@2026.3.0/ingress/origin_service.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/origin_service.go)
- Package: ingress
- Module group: ingress

## Behavioral Responsibility

Ingress matching and origin dispatch behavior.

## Entry Points

- (*unixSocketPath) String() string (line 49)
- (unixSocketPath) MarshalJSON() ([]byte, error) (line 66)
- (*httpService) String() string (line 88)
- (httpService) MarshalJSON() ([]byte, error) (line 92)
- (*rawTCPService) String() string (line 105)
- (rawTCPService) MarshalJSON() ([]byte, error) (line 113)
- (*tcpOverWSService) String() string (line 172)
- (tcpOverWSService) MarshalJSON() ([]byte, error) (line 195)
- (*socksProxyOverWSService) String() string (line 203)
- (socksProxyOverWSService) MarshalJSON() ([]byte, error) (line 207)
- (*helloWorld) String() string (line 218)
- (helloWorld) MarshalJSON() ([]byte, error) (line 247)
- (*statusCode) String() string (line 270)
- (statusCode) MarshalJSON() ([]byte, error) (line 282)
- NewWarpRoutingService(config WarpRoutingConfig, writeTimeout time.Duration) *WarpRoutingService (line 292)
- (*ManagementService) String() string (line 320)
- (ManagementService) MarshalJSON() ([]byte, error) (line 324)
- NewManagementRule(management *management.ManagementService) Rule (line 328)
- (*NopReadCloser) Read(buf []byte) (int, error) (line 338)
- (*NopReadCloser) Close() error (line 342)
- (MockOriginHTTPService) RoundTrip(req *http.Request) (*http.Response, error) (line 397)
- (MockOriginHTTPService) String() string (line 401)
- (MockOriginHTTPService) MarshalJSON() ([]byte, error) (line 409)

## Internal Function Surface

- (*unixSocketPath) start(log*zerolog.Logger, _ <-chan struct{}, cfg OriginRequestConfig) error (line 57)
- (*httpService) start(log*zerolog.Logger, _ <-chan struct{}, cfg OriginRequestConfig) error (line 77)
- (*rawTCPService) start(_*zerolog.Logger,_ <-chan struct{}, _ OriginRequestConfig) error (line 109)
- newTCPOverWSService(url *url.URL)*tcpOverWSService (line 131)
- newBastionService() *tcpOverWSService (line 148)
- newSocksProxyOverWSService(accessPolicy *ipaccess.Policy)*socksProxyOverWSService (line 154)
- addPortIfMissing(uri *url.URL, port int) (line 164)
- (*tcpOverWSService) start(log*zerolog.Logger, _ <-chan struct{}, cfg OriginRequestConfig) error (line 184)
- (*socksProxyOverWSService) start(log*zerolog.Logger, _ <-chan struct{}, cfg OriginRequestConfig) error (line 199)
- (*helloWorld) start(log*zerolog.Logger, shutdownC <-chan struct{}, cfg OriginRequestConfig) error (line 223)
- newStatusCode(status int) statusCode (line 261)
- newDefaultStatusCode(log *zerolog.Logger) statusCode (line 266)
- (*statusCode) start(log*zerolog.Logger, _ <-chan struct{}, cfg OriginRequestConfig) error (line 274)
- newManagementService(managementProxy HTTPLocalProxy) *ManagementService (line 310)
- (*ManagementService) start(log*zerolog.Logger, _ <-chan struct{}, cfg OriginRequestConfig) error (line 316)
- newHTTPTransport(service OriginService, cfg OriginRequestConfig, log *zerolog.Logger) (*http.Transport, error) (line 346)
- (MockOriginHTTPService) start(log *zerolog.Logger, _ <-chan struct{}, cfg OriginRequestConfig) error (line 405)

## Input Contract

- HTTP requests
- func-param:_ *zerolog.Logger
- func-param:_ <-chan struct{}
- func-param:_ OriginRequestConfig
- func-param:accessPolicy *ipaccess.Policy
- func-param:buf []byte
- func-param:cfg OriginRequestConfig
- func-param:config WarpRoutingConfig
- func-param:log *zerolog.Logger
- func-param:management *management.ManagementService
- func-param:managementProxy HTTPLocalProxy
- func-param:port int
- func-param:req *http.Request
- func-param:service OriginService
- func-param:shutdownC <-chan struct{}
- func-param:status int
- func-param:uri *url.URL
- func-param:url *url.URL
- func-param:writeTimeout time.Duration

## Output Contract

- return:*ManagementService
- return:*WarpRoutingService
- return:*http.Response
- return:*http.Transport
- return:*socksProxyOverWSService
- return:*tcpOverWSService
- return:Rule
- return:[]byte
- return:error
- return:int
- return:statusCode
- return:string
- stdout/stderr or structured logs

## Side Effects and State Transitions

- network I/O

## Branching and Failure Semantics

- Branch density: if=12, switch=2, select=0
- error-return paths
- fallback/default branches

## Import and Dependency Surface

- context
- crypto/tls
- encoding/json
- fmt
- github.com/cloudflare/cloudflared/hello
- github.com/cloudflare/cloudflared/ipaccess
- github.com/cloudflare/cloudflared/management
- github.com/cloudflare/cloudflared/socks
- github.com/cloudflare/cloudflared/tlsconfig
- github.com/pkg/errors
- github.com/rs/zerolog
- io
- net
- net/http
- net/url
- strconv
- time

## Go-Impl Flow (Intra-file)

```mermaid
flowchart TD
    F1["*unixSocketPath.String"]
    F2["*unixSocketPath.start"]
    F3["unixSocketPath.MarshalJSON"]
    F4["*httpService.start"]
    F5["*httpService.String"]
    F6["httpService.MarshalJSON"]
    F7["*rawTCPService.String"]
    F8["*rawTCPService.start"]
    F9["rawTCPService.MarshalJSON"]
    F10["newTCPOverWSService"]
    F11["newBastionService"]
    F12["newSocksProxyOverWSService"]
    F13["addPortIfMissing"]
    F14["*tcpOverWSService.String"]
    F15["*tcpOverWSService.start"]
    F16["tcpOverWSService.MarshalJSON"]
    F17["*socksProxyOverWSService.start"]
    F18["*socksProxyOverWSService.String"]
    F19["socksProxyOverWSService.MarshalJSON"]
    F20["*helloWorld.String"]
    F21["*helloWorld.start"]
    F22["helloWorld.MarshalJSON"]
    F23["newStatusCode"]
    F24["newDefaultStatusCode"]
    F25["*statusCode.String"]
    F26["*statusCode.start"]
    F27["statusCode.MarshalJSON"]
    F28["NewWarpRoutingService"]
    F29["newManagementService"]
    F30["*ManagementService.start"]
    F31["*ManagementService.String"]
    F32["ManagementService.MarshalJSON"]
    F33["NewManagementRule"]
    F34["*NopReadCloser.Read"]
    F35["*NopReadCloser.Close"]
    F36["newHTTPTransport"]
    F37["MockOriginHTTPService.RoundTrip"]
    F38["MockOriginHTTPService.String"]
    F39["MockOriginHTTPService.start"]
    F40["MockOriginHTTPService.MarshalJSON"]
    F2 --> F36
    F4 --> F36
    F10 --> F13
    F33 --> F29
```

## Accuracy Notes

- Generated from Go AST parsing and source text pattern extraction.
- Source link is authoritative for disputed semantics; keep this atom synchronized with the linked file.

## Rust Porting Notes

- **Origin service hierarchy**: Go interface `OriginService` with 7+ concrete implementations → Rust `enum OriginService { Http(HttpService), RawTcp(RawTcpService), TcpOverWs(TcpOverWsService), SocksProxy(SocksProxyService), HelloWorld(HelloWorldService), StatusCode(u16), Management(ManagementService) }` for exhaustive matching.
- **JSON serialization**: Per-type `MarshalJSON` methods → `#[derive(Serialize)]` with `#[serde(tag = "type")]` for tagged enum serialization.
- **Unix socket**: `unixSocketPath` type → `std::os::unix::net::UnixStream` behind `#[cfg(unix)]`; on Windows, return an error.
- **TLS client config**: `tlsconfig` for HTTPS origins → `rustls::ClientConfig` with custom CA and origin server name.
- **Hello world server**: Built-in test origin → `axum` or `hyper` server serving static HTML on localhost.
- **SOCKS proxy**: `socks.StreamHandler` → `fast-socks5` or custom SOCKS5 implementation over async streams.
- **IP access rules**: `ipaccess.Policy` for origin access control → IP range matching via `ipnet` crate.
- **Quirk — NopReadCloser**: Adapter returning empty reads → `std::io::empty()` or `tokio::io::empty()` in async context.
- **Quirk — MockOriginHTTPService**: Test mock implementing `RoundTrip` → `tower::Service<Request<Body>>` mock for testing.
