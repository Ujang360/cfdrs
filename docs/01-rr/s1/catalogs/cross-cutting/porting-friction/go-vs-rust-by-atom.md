# Porting Friction — Go vs Rust by Atom

> Part of the [Porting Friction Behavior Catalog](README.md).
>
> This file aggregates the `## Rust Porting Notes` from all 79 atoms referenced
> by this catalog, organized by Go package. Each entry shows the concrete Go
> construct and its Rust equivalent — a per-atom "A vs B" reference for the port.

## carrier

### [carrier.go](../../../atoms/carrier/carrier.md)

| Go construct | Rust equivalent |
| --- | --- |
| `StdinoutStream` wrapping OS stdin/stdout (`io.Reader + io.Writer`) | `tokio::io::stdin()` / `tokio::io::stdout()` with `AsyncRead + AsyncWrite` |
| `StartForwarder` dials origin then pipes bidirectionally | `tokio-tungstenite` for WebSocket upgrade, then `tokio::io::copy_bidirectional` |
| `<-chan struct{}` shutdown channel | `tokio_util::sync::CancellationToken` |
| `BuildAccessRequest` constructs HTTP request with token headers | `reqwest::Request` builder or manual `http::Request<Body>` construction |
| `SetBastionDest` / `ResolveBastionDest` header manipulation | `http::HeaderMap` typed accessors |
| `crypto/tls` dialing | `rustls` with custom `ServerName` |
| `select` with 2 cases (shutdown or error) | `tokio::select!` on cancellation token and error channel |

## cfapi

### [base_client.go](../../../atoms/cfapi/base_client.md)

| Go construct | Rust equivalent |
| --- | --- |
| `fetchExhaustively[T any]` Go generic pagination | `async fn fetch_all<T: DeserializeOwned>(…) -> Result<Vec<T>>` |
| `golang.org/x/net/http2` transport | `reqwest::Client` (h2 enabled by default) or `hyper` with h2 |
| Nested `result`/`errors`/`messages` JSON envelope | `#[derive(Deserialize)] struct ApiResponse<T> { result: T, errors: Vec<ApiError> }` |
| 23 if-branches for pagination + error handling | Async stream or `loop { … break }` with `?` |

## cmd/cloudflared

### [generic_service.go](../../../atoms/cmd/cloudflared/generic_service.md)

| Go construct | Rust equivalent |
| --- | --- |
| `runApp()` + platform-specific `installGenericService()` | `#[cfg(target_os = "...")]` conditional compilation |

### [linux_service.go](../../../atoms/cmd/cloudflared/linux_service.md)

| Go construct | Rust equivalent |
| --- | --- |
| `installSystemd()` renders `.service` unit template | `askama` or `format!` template rendering; write to `/etc/systemd/system/` |
| Sysv fallback when systemd unavailable | Runtime detection via `Path::new("/run/systemd/system").exists()` |
| `copyFile()` + permission setting | `std::fs::copy()` + `std::os::unix::fs::PermissionsExt` |

### [macos_service.go](../../../atoms/cmd/cloudflared/macos_service.md)

| Go construct | Rust equivalent |
| --- | --- |
| `installLaunchd()` renders `.plist` XML template | `plist` crate for structured XML, or `askama` template |
| `go-homedir` with root user check | `dirs::home_dir()` + `nix::unistd::getuid().is_root()` |

### [windows_service.go](../../../atoms/cmd/cloudflared/windows_service.md)

| Go construct | Rust equivalent |
| --- | --- |
| `golang.org/x/sys/windows/svc` service control | `windows-service` crate with `ServiceControlHandler` and `ServiceMain` |
| `svc/mgr` for service install/remove | `windows-service::service_manager::ServiceManager` API |
| `svc/eventlog` | `eventlog` crate or `tracing-eventlog` subscriber |
| Entire file `//go:build windows` | `#[cfg(target_os = "windows")]` module |
| `unsafe` + `syscall` for Windows handles | `windows-sys` crate typed bindings; minimize `unsafe` blocks |
| `select` on `svc.ChangeRequest` channel | `tokio::sync::mpsc::Receiver` in a `tokio::select!` loop |

### [tunnel/cmd.go](../../../atoms/cmd/cloudflared/tunnel/cmd.md)

| Go construct | Rust equivalent |
| --- | --- |
| `urfave/cli/v2` commands and flags (1000+ line flag block) | `clap` derive macros with `#[command(subcommand)]` enums |
| `sync.WaitGroup` + `<-chan error` + `<-chan struct{}` shutdown | `tokio::task::JoinSet` or structured `tokio::select!` on mpsc receivers |
| `go-systemd/v22/daemon.SdNotify` | `sd-notify` crate: `notify(false, &[NotifyState::Ready])` |
| `writePidFile` | `std::fs::write` with a `Drop` guard for cleanup |
| `stdinControl` goroutine reading stdin | `tokio::io::BufReader::new(tokio::io::stdin())` with line-based parsing |
| `chan supervisor.ReconnectSignal` | `tokio::sync::mpsc::Sender<ReconnectSignal>` |
| `go-homedir` | `dirs::home_dir()` |
| `grace/gracenet` FD-passing | `listenfd` crate or manual `FromRawFd` on systemd-passed sockets |
| `sentry-go` | `sentry` crate with `sentry::capture_error` |

### [tunnel/subcommands.go](../../../atoms/cmd/cloudflared/tunnel/subcommands.md)

| Go construct | Rust equivalent |
| --- | --- |
| `urfave/cli/v2` subcommand registration | `clap` derive macros with `#[command(subcommand)]` enum variants |
| `ParseToken` base64-decodes tunnel token | `base64::engine::general_purpose::STANDARD.decode()` + `serde_json::from_slice` |
| `cfapi` client for tunnel CRUD | `reqwest::Client` with typed request/response structs |
| `golang.org/x/net/idna` hostname normalization | `idna` crate for IDNA2008 processing |
| `gopkg.in/yaml.v3` | `serde_yaml` for config round-tripping |
| `text/tabwriter` for formatted CLI output | `tabwriter` or `comfy-table` crate |
| `regexp` for hostname validation | `regex` crate; compile lazily with `std::sync::LazyLock` |
| 88 if-branches (highest in codebase) | Per-subcommand handler functions with enum dispatch |

## config

### [configuration.go](../../../atoms/config/configuration.md)

| Go construct | Rust equivalent |
| --- | --- |
| `FindDefaultConfigPath` platform-conditional path search | `dirs::config_dir()` + platform fallbacks via `#[cfg(target_os)]` |
| `gopkg.in/yaml.v3` two-pass decode | `serde_yaml::from_reader` with `#[derive(Deserialize)]` |
| `CustomDuration` (JSON as integer seconds, YAML as Go duration string) | `#[serde(deserialize_with)]` for dual-format handling — **highest-risk serialization asymmetry** |
| `configFileSettings` wrapping YAML for `urfave/cli` | `clap` config support via `config` crate or manual layered config |
| `ValidateUnixSocket` | `std::os::unix::net::UnixListener::bind` probe; `#[cfg(unix)]` |
| `ValidateUrl` with scheme/host normalization | `url::Url::parse` with custom validation |
| `GetConfiguration` singleton via `sync.Once` | `std::sync::OnceLock<Configuration>` |

### [manager.go](../../../atoms/config/manager.md)

| Go construct | Rust equivalent |
| --- | --- |
| `watcher.Notifier` interface for config reload | `notify::RecommendedWatcher` with async channel receiver |
| `gopkg.in/yaml.v3` | `serde_yaml::from_str()` |

## connection

### [control.go](../../../atoms/connection/control.md)

| Go construct | Rust equivalent |
| --- | --- |
| `io.ReadWriteCloser` stream | `tokio::io::AsyncRead + AsyncWrite + Unpin` or boxed trait object |
| `ConnectedFuse` one-shot notification | `tokio::sync::Notify` or `Arc<AtomicBool>` with `Notify` wakeup |
| `<-chan struct{}` graceful-shutdown | `tokio_util::sync::CancellationToken` |
| `select` on context + shutdown | `tokio::select!` with cancellation token |
| `tunnelrpc.RegistrationClient` Cap'n Proto client | Generated Rust Cap'n Proto client via `capnpc-rust` |
| `registerClientFunc` function-typed field | `Box<dyn Fn(...) -> ... + Send + Sync>` or async trait method |
| Timeout-guarded unregistration | `tokio::time::timeout(duration, future)` |

### [errors.go](../../../atoms/connection/errors.md)

| Go construct | Rust equivalent |
| --- | --- |
| Seven Go error types (`DupConnRegister`, `EdgeQuicDial`, etc.) | `#[derive(thiserror::Error)] enum ConnectionError` with per-category variants |
| `EdgeQuicDialError.Unwrap()` | `#[source]` attribute for auto `Error::source()` |
| `errors.Is` / `errors.As` matching | `match` on enum variants or `downcast_ref` with `anyhow` |
| `DupConnRegisterTunnelError` as string-only error | Variant with message field rather than bare `String` |

### [header.go](../../../atoms/connection/header.md)

| Go construct | Rust equivalent |
| --- | --- |
| `SerializeHeaders` encodes HTTP headers as base64 key-value pairs | `base64::engine::general_purpose::STANDARD.encode`; **wire format must match exactly** |
| `DeserializeHeaders` splits and decodes | `base64::engine::general_purpose::STANDARD.decode` |
| `IsControlResponseHeader` prefix check | `str::starts_with` on `http::HeaderName` |
| `IsWebsocketClientHeader` string matching | `http::header::HeaderName` type-safe comparison |

### [http2.go](../../../atoms/connection/http2.md)

| Go construct | Rust equivalent |
| --- | --- |
| `http.Server` with `http2` transport | `hyper::server::conn::http2::Builder` over TLS `TcpStream` |
| `http2RespWriter` implementing `http.ResponseWriter` + `Hijacker` | Custom struct wrapping `hyper::body::Sender` with header/trailer support |
| Go `Hijacker` returns raw `net.Conn` | `hyper::upgrade::on()` to extract underlying stream |
| Config update stream multiplexed over HTTP/2 | Dedicated `h2` stream or management endpoint |
| `runtime/debug.Stack()` on panic | `std::panic::catch_unwind(AssertUnwindSafe(…))` |

### [observer.go](../../../atoms/connection/observer.md)

| Go construct | Rust equivalent |
| --- | --- |
| `EventSink` Go interface (4 methods) | `trait EventSink: Send + Sync` with async methods |
| Nil channel semantics (send to nil blocks forever) | `Option<Sender<T>>` with conditional send |
| `connection.Event` interface hierarchy | Rust enum `ConnectionEvent { Connected(…), Disconnected(…), … }` |
| `sync.Mutex`-guarded observer list | `Arc<RwLock<Vec<Box<dyn EventSink>>>>` |

### [quic_connection.go](../../../atoms/connection/quic_connection.md)

| Go construct | Rust equivalent |
| --- | --- |
| `quic-go.Connection` | `quinn::Connection` with `accept_bi()` / `accept_uni()` |
| `errgroup.Group` for concurrent serving | `tokio::task::JoinSet` with first-error propagation |
| `httpResponseAdapter` implementing `http.ResponseWriter` | Custom struct wrapping `quinn::SendStream` |
| `streamReadWriteAcker.AckConnection` | Async `write_all` on `SendStream` |
| `sync/atomic` for config version tracking | `std::sync::atomic::AtomicI32` |
| `rpcTimeout` / `streamWriteTimeout` | `tokio::time::timeout` wrapping each call |
| `nopCloserReadWriter` adapter (no-op `Close()`) | Unnecessary if type doesn't need `AsyncWrite + Close`; else newtype with no-op `Drop` |

## credentials

### [origin_cert.go](../../../atoms/credentials/origin_cert.md)

| Go construct | Rust equivalent |
| --- | --- |
| `[]byte` cert/key handling | `Vec<u8>` / `&[u8]` with `rustls::pki_types` for typed cert containers |

## datagramsession

### [manager.go](../../../atoms/datagramsession/manager.md)

| Go construct | Rust equivalent |
| --- | --- |
| `io.ReadWriteCloser` session origin | `tokio::io::AsyncRead + AsyncWrite` trait object |
| `sync.Map` for concurrent session tracking | `DashMap<SessionId, SessionEntry>` |
| `select` on context + unregister channel | `tokio::select!` with cancellation token |

### [metrics.go](../../../atoms/datagramsession/metrics.md)

| Go construct | Rust equivalent |
| --- | --- |
| `init()` → `prometheus.MustRegister(activeUDPSessions, totalUDPSessions)` | `static METRIC: LazyLock<IntGauge> = LazyLock::new(…)` |

## diagnostic

### [network/collector_unix.go](../../../atoms/diagnostic/network/collector_unix.md)

| Go construct | Rust equivalent |
| --- | --- |
| `//go:build unix` | `#[cfg(unix)]` |

### [network/collector_windows.go](../../../atoms/diagnostic/network/collector_windows.md)

| Go construct | Rust equivalent |
| --- | --- |
| `//go:build windows` | `#[cfg(target_os = "windows")]` |

### [system_collector_linux.go](../../../atoms/diagnostic/system_collector_linux.md)

| Go construct | Rust equivalent |
| --- | --- |
| `//go:build linux` | `#[cfg(target_os = "linux")]` |

### [system_collector_windows.go](../../../atoms/diagnostic/system_collector_windows.md)

| Go construct | Rust equivalent |
| --- | --- |
| `//go:build windows` | `#[cfg(target_os = "windows")]` |

## edgediscovery

### [dial.go](../../../atoms/edgediscovery/dial.md)

| Go construct | Rust equivalent |
| --- | --- |
| `context.Context` + `defer conn.Close()` | `CancellationToken` + `Drop` impl or scope guard |
| `crypto/tls.Dial` | `tokio_rustls::TlsConnector::connect()` |

## fips

### [fips.go](../../../atoms/fips/fips.md)

| Go construct | Rust equivalent |
| --- | --- |
| `//go:build fips` + `import _ "crypto/tls/fipsonly"` | `#[cfg(feature = "fips")]` + conditional `rustls` with FIPS-compliant backend |
| `IsFipsEnabled()` returns `true` | `const FIPS_ENABLED: bool = true` under `#[cfg(feature = "fips")]` |

### [nofips.go](../../../atoms/fips/nofips.md)

| Go construct | Rust equivalent |
| --- | --- |
| `//go:build !fips` + `IsFipsEnabled()` returns `false` | Default `const FIPS_ENABLED: bool = false` |

## ingress

### [icmp_darwin.go](../../../atoms/ingress/icmp_darwin.md)

| Go construct | Rust equivalent |
| --- | --- |
| `//go:build darwin` | `#[cfg(target_os = "macos")]` |

### [icmp_generic.go](../../../atoms/ingress/icmp_generic.md)

| Go construct | Rust equivalent |
| --- | --- |
| `//go:build !darwin && !linux && (!windows \|\| !cgo)` | `#[cfg(not(any(unix, windows)))]` |

### [icmp_linux.go](../../../atoms/ingress/icmp_linux.md)

| Go construct | Rust equivalent |
| --- | --- |
| `//go:build linux` | `#[cfg(target_os = "linux")]` |

### [icmp_posix.go](../../../atoms/ingress/icmp_posix.md)

| Go construct | Rust equivalent |
| --- | --- |
| `//go:build darwin \|\| linux` | `#[cfg(unix)]` |

### [icmp_windows.go](../../../atoms/ingress/icmp_windows.md)

| Go construct | Rust equivalent |
| --- | --- |
| `//go:build windows` | `#[cfg(target_os = "windows")]` |

### [icmp_metrics.go](../../../atoms/ingress/icmp_metrics.md)

| Go construct | Rust equivalent |
| --- | --- |
| `init()` → `prometheus.MustRegister(icmpRequestsTotal, icmpRequestErrors)` | `static METRIC: LazyLock<IntCounter> = LazyLock::new(…)` |

### [origin_dialer.go](../../../atoms/ingress/origin_dialer.md)

| Go construct | Rust equivalent |
| --- | --- |
| `net.Conn` implicit interface | `tokio::net::TcpStream` or `Box<dyn AsyncRead + AsyncWrite + Unpin + Send>` |
| `context.Context` first-param | `CancellationToken` or `tokio::time::timeout` |

### [origin_icmp_proxy.go](../../../atoms/ingress/origin_icmp_proxy.md)

| Go construct | Rust equivalent |
| --- | --- |
| `ICMPRouterServer` interface (per-platform impls) | `trait IcmpRouter: Send + Sync` with `#[cfg]`-gated impls |

### [origin_proxy.go](../../../atoms/ingress/origin_proxy.md)

| Go construct | Rust equivalent |
| --- | --- |
| Multiple `RoundTrip` implementations per origin type | `trait OriginProxy { async fn proxy(&self, req: Request) -> Result<Response>; }` with per-type impls |
| Dynamic TLS config with hostname-based SNI | `tokio_rustls::TlsConnector` with per-request `ServerName` |
| 7 if-branches + 1 switch for protocol dispatch | `match` on enum variant |

### [origin_service.go](../../../atoms/ingress/origin_service.md)

| Go construct | Rust equivalent |
| --- | --- |
| `OriginService` interface — 9 implicit implementors | `enum OriginService { UnixSocket(…), Http(…), RawTcp(…), … }` or `dyn OriginService` |
| `helloWorld` embeds `httpService` (method promotion) | Store as field + manual delegation |
| `WarpRoutingService` embeds interface field | `inner: Box<dyn StreamOriginProxy>` + delegate |
| `newHTTPTransport` type switch on `service.(type)` | `match service { OriginService::UnixSocket(…) => …, _ => … }` |
| `NopReadCloser` (returns EOF) | `impl Read` that returns `Ok(0)` |

### [origins/dns.go](../../../atoms/ingress/origins/dns.md)

| Go construct | Rust equivalent |
| --- | --- |
| `net.Conn` implicit interface for DNS proxy | `tokio::net::TcpStream` or `tokio::net::UdpSocket` |
| `context.Context` for DNS resolution | `CancellationToken` + `tokio::time::timeout` |

## logger

### [console.go](../../../atoms/logger/console.md)

| Go construct | Rust equivalent |
| --- | --- |
| `consoleWriter` implicitly satisfying `io.Writer` | `impl Write` for `ConsoleWriter` |
| `go-colorable` for terminal color detection | `tracing_subscriber::fmt::SubscriberBuilder::with_ansi()` |

### [create.go](../../../atoms/logger/create.md)

| Go construct | Rust equivalent |
| --- | --- |
| `init()` setting `zerolog.TimestampFunc` to UTC | Configure at logger construction time; no global mutation |
| `sync.Once` for file writer singleton | `std::sync::OnceLock` or `LazyLock` |
| `resilientMultiWriter` implicitly satisfying `io.Writer` + `zerolog.LevelWriter` | `impl Write` + custom `LevelWrite` trait |
| `lumberjack` for log rotation | `tracing_appender::rolling::daily()` or `file-rotate` crate |
| `urfave/cli` for log level flags | `clap` derive struct |
| `go-colorable` | `tracing_subscriber::fmt::with_ansi()` |

## management

### [events.go](../../../atoms/management/events.md)

| Go construct | Rust equivalent |
| --- | --- |
| `LogEventType` / `LogLevel` iota enums | `#[derive(Serialize, Deserialize)]` enums with `#[serde(rename_all = "lowercase")]` |
| `IntoClientEvent[T]` / `IntoServerEvent[T]` Go generics | Rust enum variants with typed payloads |
| `nhooyr.io/websocket` | `tokio-tungstenite::Message` or `axum::extract::ws::Message` |
| `json-iterator/go` | `serde_json` |
| `IsClosed` / `AsClosed` error matching | `match` on `tungstenite::Error::ConnectionClosed` |
| `WriteEvent` accepts `any` | `ServerEvent` enum implementing `Serialize` |

### [logger.go](../../../atoms/management/logger.md)

| Go construct | Rust equivalent |
| --- | --- |
| `sync.Mutex` guarding session writer map | `Arc<RwLock<HashMap<SessionId, Sender<LogEvent>>>>` or `DashMap` |
| `zerolog.LevelWriter` interface | Custom `tracing_subscriber::Layer` with per-level dispatch |

## metrics

### [metrics.go](../../../atoms/metrics/metrics.md)

| Go construct | Rust equivalent |
| --- | --- |
| `facebookarchive/grace/gracenet` for socket inheritance | `listenfd` crate or `systemfd` |
| `net/http/pprof` | `pprof-rs` crate or custom Prometheus endpoint |
| `context.WithCancel` for metrics server lifecycle | `tokio::select!` with `CancellationToken` |

## orchestration

### [config.go](../../../atoms/orchestration/config.md)

| Go construct | Rust equivalent |
| --- | --- |
| Ingress rule JSON marshaling | `#[derive(Serialize, Deserialize)]` with `serde_json` |

### [orchestrator.go](../../../atoms/orchestration/orchestrator.md)

| Go construct | Rust equivalent |
| --- | --- |
| `sync.RWMutex`-guarded `currentConfig` | `arc_swap::ArcSwap<Config>` for lock-free reads |
| `sync/atomic` int32 version counter | `std::sync::atomic::AtomicI32` with `Ordering::SeqCst` |
| `GetOriginProxy` returns proxy behind read lock | `ArcSwap::load()` → `Arc<dyn OriginProxy>` without contention |
| Nil interface check `if proxy == nil` | `Option<Box<dyn OriginProxy>>` |
| `encoding/json.Unmarshal` for config updates | `serde_json::from_slice` |

## packet

### [decoder.go](../../../atoms/packet/decoder.md)

| Go construct | Rust equivalent |
| --- | --- |
| `gopacket.NewPacket` + `layers.IPv4/IPv6/ICMPv4` | `etherparse::SlicedPacket::from_ip()` or `pnet_packet` crate |
| `packet.Layer(layers.LayerTypeICMPv4)` | `match` on parsed packet enum variant |

### [funnel.go](../../../atoms/packet/funnel.md)

| Go construct | Rust equivalent |
| --- | --- |
| `sync.Map` + `sync/atomic` counters + goroutine cleanup | `DashMap<FunnelId, FunnelEntry>` with `tokio::spawn` idle-timeout eviction |
| `context.WithCancel` per funnel | Per-funnel `CancellationToken` |

### [packet.go](../../../atoms/packet/packet.md)

| Go construct | Rust equivalent |
| --- | --- |
| `gopacket/layers` for packet building | `etherparse::PacketBuilder` or `pnet_packet::icmp` |
| `x/net/icmp` `Message.Marshal()` | Manual ICMP checksum + serialization or `pnet` crate |

## quic

### [datagram.go](../../../atoms/quic/datagram.md)

| Go construct | Rust equivalent |
| --- | --- |
| `DatagramMuxer` channel-based session dispatch | `quinn::Connection::send_datagram()` / `read_datagram()` + `tokio::sync::mpsc` |
| `ServeReceive` goroutine loop with `select` on context | `tokio::select!` over `connection.read_datagram()` and `cancellation_token.cancelled()` |
| UUID suffix/extract on byte slices | `uuid::Uuid` with `as_bytes()` / `from_slice()` |
| `chan<- *packet.Session` | `mpsc::Sender<SessionPacket>::send().await` |

### [datagramv2.go](../../../atoms/quic/datagramv2.md)

| Go construct | Rust equivalent |
| --- | --- |
| `RawPacket`, `TracedPacket`, `TracingSpanPacket` type hierarchy | `enum DatagramPacket { Raw(Bytes), Traced {…}, TracingSpan {…} }` |
| `DatagramV2Type` discriminator enum | `#[repr(u8)] enum DatagramV2Type` with `TryFrom<u8>` |
| 3 `select` blocks | `tokio::select!` with cancellation safety annotations |
| `switch v := pkt.(type)` | `match` on enum variants |
| Struct embedding for shared fields | Shared fields in enum variant data |

### [param_unix.go](../../../atoms/quic/param_unix.md)

| Go construct | Rust equivalent |
| --- | --- |
| `//go:build unix` | `#[cfg(unix)]` or `#[cfg(target_family = "unix")]` |

### [param_windows.go](../../../atoms/quic/param_windows.md)

| Go construct | Rust equivalent |
| --- | --- |
| `//go:build windows` | `#[cfg(target_os = "windows")]` |

### [safe_stream.go](../../../atoms/quic/safe_stream.md)

| Go construct | Rust equivalent |
| --- | --- |
| `SafeStreamCloser` wrapping QUIC stream | Struct implementing `AsyncRead + AsyncWrite + Drop` over `quinn::SendStream` / `RecvStream` |
| `sync/atomic` close-state flag | `AtomicBool` with `Ordering::Acquire` / `Release` |
| Implicit `net.Conn` (`io.Reader + Writer + Closer`) | Explicit `impl AsyncRead`, `impl AsyncWrite` |
| `SetDeadline()` / `SetReadDeadline()` / `SetWriteDeadline()` | `tokio::time::timeout(duration, stream_op)` wrappers (quinn lacks per-op deadlines) |
| Error classification in `handleWriteError` | `match` on `quinn::WriteError` variants |

### [v3/session.go](../../../atoms/quic/v3/session.md)

| Go construct | Rust equivalent |
| --- | --- |
| `io.ReadWriteCloser` origin stream | `tokio::io::AsyncRead + AsyncWrite` trait object or generic bound |
| Two goroutines (`readLoop`, `writeLoop`) | `tokio::spawn` tasks in `JoinSet` or `tokio::select!` in a single `Serve` future |
| `sync/atomic` int64 last-active timestamp | `std::sync::atomic::AtomicI64` with `Ordering::Relaxed` |
| `time.Timer` reset pattern | `tokio::time::Sleep` in `Pin<Box<Sleep>>` with `reset()` |
| `sync.Once` for session teardown | `std::sync::Once` or `Drop` impl + `AtomicBool` flag |
| `DatagramConn` Go interface | Async trait: `async fn send_datagram(&self, payload: &[u8])` |
| `Migrate` swaps eyeball connection | `Arc<Mutex<DatagramConn>>` or atomic swap pattern |
| 6 `select` statements | Consolidate into fewer `tokio::select!` blocks |

## retry

### [backoffhandler.go](../../../atoms/retry/backoffhandler.md)

| Go construct | Rust equivalent |
| --- | --- |
| Exponential backoff with jitter | `backon` crate or manual `tokio::time::sleep(Duration::from_secs(base * 2^retries) + jitter)` |
| `BackoffTimer() <-chan time.Time` | `async fn backoff_sleep(&mut self)` returning `Sleep` future |
| `Backoff(ctx)` respects cancellation | `tokio::select!` on sleep + cancellation token |
| `SetGracePeriod` from current backoff | `Option<Instant>` checking `elapsed()` before retry |
| `math/rand` for jitter | `rand::thread_rng().gen_range(0..=max_jitter)` |
| `retryForever` flag | `enum RetryPolicy { MaxRetries(u32), Forever }` |

## socks

### [authenticator.go](../../../atoms/socks/authenticator.md)

| Go construct | Rust equivalent |
| --- | --- |
| `io.Reader` / `io.Writer` based SOCKS5 auth exchange | `tokio::io::AsyncReadExt::read_exact()` + `AsyncWriteExt::write_all()` |
| `NoAuthAuthenticator` / `UserPassAuthAuthenticator` implicitly satisfying `Authenticator` | `impl Authenticator for NoAuth`, `impl Authenticator for UserPass` |

### [dialer.go](../../../atoms/socks/dialer.md)

| Go construct | Rust equivalent |
| --- | --- |
| `Dialer` interface (`net.Dial` abstraction) | `trait Dialer: Send + Sync { async fn dial(&self, addr: &str) -> io::Result<TcpStream>; }` |

## stream

### [debug.go](../../../atoms/stream/debug.md)

| Go construct | Rust equivalent |
| --- | --- |
| `sync/atomic` counters for bytes read/written | `AtomicU64::fetch_add()` wrapping `AsyncRead` / `AsyncWrite` impl |
| Conditional debug logging | `tracing::trace!` with counter fields |

## supervisor

### [fuse.go](../../../atoms/supervisor/fuse.md)

| Go construct | Rust equivalent |
| --- | --- |
| `booleanFuse` with `sync.Once` semantics | `tokio::sync::Notify` or `tokio::sync::watch::channel(false)` |
| `Await()` blocks until fuse fires | `notify.notified().await` or `watch_rx.changed().await` |
| `Value()` non-blocking read | `watch_rx.borrow().clone()` or `AtomicBool::load(Ordering::Acquire)` |
| `sync.Mutex` + `sync.Cond` | `tokio::sync` primitives (avoid blocking the executor) |

### [supervisor.go](../../../atoms/supervisor/supervisor.md)

| Go construct | Rust equivalent |
| --- | --- |
| `Run` spawns per-connection goroutines via `errgroup` | `tokio::task::JoinSet` with per-connection cancellation tokens |
| `edgediscovery.Edge` address allocator | Async `EdgeDiscovery` trait with `get_addr(conn_index)` |
| `retry.BackoffHandler` | Compose with Rust `BackoffHandler` equivalent |
| `chan ReconnectSignal` | `tokio::sync::mpsc::Receiver<ReconnectSignal>` |
| `prometheus.Counter` / `Gauge` | `metrics` crate macros or `prometheus` Rust crate |
| `tunnelstate.ConnTracker` | `Arc<ConnTracker>` with atomic state transitions |
| `quic-go.EarlyListener` | `quinn::Endpoint` with server config |
| 18 if-branches + 2 select | State machine enum: `Connecting`, `Connected`, `Backoff`, `Shutdown` |

### [tunnel.go](../../../atoms/supervisor/tunnel.md)

| Go construct | Rust equivalent |
| --- | --- |
| `errgroup` goroutine management | `tokio::task::JoinSet` with first-error propagation |
| `protocolFallback` struct | `enum ProtocolState { Current(Protocol), FallenBack(Protocol) }` |
| Multiple `<-chan struct{}` channels | `tokio_util::sync::CancellationToken` tree |
| `crypto/tls` | `rustls::ClientConfig` with `webpki-roots` or custom CA store |
| `quic-go` listener/dialer | `quinn` crate with matching ALPN and transport params |
| `connIndex uint8` | Newtype `ConnIndex(u8)` |
| `sentry-go` | `sentry` crate; `debug.Stack()` → `std::backtrace::Backtrace::capture()` |
| `ipAddrFallback` per-connection retry counters | `HashMap<ConnIndex, u8>` or array-backed counter |
| `connectedFuse` one-shot | `tokio::sync::Notify` or `tokio::sync::watch` |
| `serveTunnel` returns `(error, bool)` for recoverability | `Result<(), TunnelError>` where variants encode recoverability |
| 34 if-branches + 4 switches | Match-based dispatch on typed enums |

## tlsconfig

### [certreloader.go](../../../atoms/tlsconfig/certreloader.md)

| Go construct | Rust equivalent |
| --- | --- |
| `sync.RWMutex`-guarded cert/key pair | `Arc<RwLock<Arc<rustls::sign::CertifiedKey>>>` with `ResolvesServerCert` |
| Cert file change detection | `notify` crate watcher triggering reload |
| `x509.NewCertPool` + `AppendCertsFromPEM` | `rustls::RootCertStore::add_parsable_certificates()` |
| `github.com/pkg/errors.Wrap` | `.context("msg")?` with `anyhow` |

## token

### [launch_browser_darwin.go](../../../atoms/token/launch_browser_darwin.md)

| Go construct | Rust equivalent |
| --- | --- |
| `//go:build darwin` + `exec.Command("open", url)` | `#[cfg(target_os = "macos")]` + `std::process::Command::new("open").arg(url)` or `open` crate |

### [launch_browser_other.go](../../../atoms/token/launch_browser_other.md)

| Go construct | Rust equivalent |
| --- | --- |
| Catch-all platform fallback | `#[cfg(not(any(target_os = "macos", target_os = "linux", target_os = "windows")))]` + `open` crate |

### [launch_browser_windows.go](../../../atoms/token/launch_browser_windows.md)

| Go construct | Rust equivalent |
| --- | --- |
| `//go:build windows` + `syscall.SysProcAttr{HideWindow: true}` | `#[cfg(target_os = "windows")]` + `CommandExt::creation_flags(CREATE_NO_WINDOW)` |
| `cmd /c start url` | `std::process::Command::new("cmd").args(["/c", "start", url])` |

## tracing

### [client.go](../../../atoms/tracing/client.md)

| Go construct | Rust equivalent |
| --- | --- |
| `sync.Mutex` protecting in-memory span list | `Arc<Mutex<Vec<SpanData>>>` or `parking_lot::Mutex` |
| `InMemory` + `Noop` pair (two interface impls) | `enum TracingClient { InMemory(InMemoryCollector), Noop }` with `match` dispatch |
| `otlp/trace/v1` + `google.golang.org/protobuf` | `opentelemetry_proto` crate or `prost`-generated types |

### [identity.go](../../../atoms/tracing/identity.md)

| Go construct | Rust equivalent |
| --- | --- |
| Manual byte encoding of trace/span IDs (`encoding.BinaryMarshaler`) | `opentelemetry::trace::TraceId::from_bytes()` / `to_bytes()` or `[u8; 16]` |
| `[]byte` serialization | `impl Serialize` or manual `to_vec()` |

### [tracing.go](../../../atoms/tracing/tracing.md)

| Go construct | Rust equivalent |
| --- | --- |
| `go.opentelemetry.io/otel` SDK init with Jaeger propagator | `opentelemetry_sdk::trace::TracerProvider` + `opentelemetry_otlp::SpanExporter` + `opentelemetry_jaeger_propagator` |
| `init()` global tracer registration | `opentelemetry::global::set_tracer_provider()` called once |
| HTTP header extraction via `propagation.Extract` | `opentelemetry::global::get_text_map_propagator().extract(&HeaderExtractor(headers))` |
| `TracedHTTPRequest` embeds `*http.Request` + `*cfdTracer` | Struct with two fields + implement both traits |
| `TracedContext` embeds `context.Context` + `*cfdTracer` | Struct with `CancellationToken` + `Span` fields |

## tunnelrpc

### [metrics/metrics.go](../../../atoms/tunnelrpc/metrics/metrics.md)

| Go construct | Rust equivalent |
| --- | --- |
| `init()` → `prometheus.MustRegister(...)` for 5+ RPC metrics | `std::sync::LazyLock` or register in a setup function called from `main` |
| `ObserveServerHandler` wraps RPC handler with latency/error tracking | `tower::Layer` middleware or manual `Instant::now()` + `elapsed()` |
| `NewClientOperationLatencyObserver` | `prometheus::Histogram::start_timer()` or `metrics::histogram!` |
| Handler/method name string labels | `&'static str` constants for label values |

### [proto/quic_metadata_protocol.capnp.go](../../../atoms/tunnelrpc/proto/quic_metadata_protocol.capnp.md)

| Go construct | Rust equivalent |
| --- | --- |
| Generated from `.capnp` schema via `capnpc-go` | Regenerate via `capnpc-rust` in `build.rs`; **do not hand-port** |
| Segment-based memory model (arena-allocated) | `capnp::message::Builder` / `Reader` |
| `HasField()` optional checks | `reader.has_field()` in `capnp-rust` |
| `init()` schema registration | Not needed; `capnp-rust` uses compile-time schema embedding |

### [proto/tunnelrpc.capnp.go](../../../atoms/tunnelrpc/proto/tunnelrpc.capnp.md)

| Go construct | Rust equivalent |
| --- | --- |
| `.capnp` schema compiled via `capnpc-go` | Compile via `capnpc-rust` in `build.rs` using `capnpc::CompilerCommand` |
| 300+ generated entry points | `capnp` codegen produces equivalent reader/builder types |
| `TunnelServer`, `RegistrationServer`, `SessionManager`, `ConfigurationManager` RPC interfaces | Generated Rust server traits and client stubs |
| Promise pipelining | `capnp-rpc` supports `request.send().pipeline` |
| **Identical `.capnp` schema required** for wire compatibility | Use the **same schema file** — Rust and Go must produce wire-compatible messages |

### [quic/cloudflared_client.go](../../../atoms/tunnelrpc/quic/cloudflared_client.md)

| Go construct | Rust equivalent |
| --- | --- |
| `zombiezen.com/go/capnproto2/rpc` | `capnp-rpc` crate with generated client stubs |
| `io.ReadWriteCloser` as RPC transport | `tokio::io::AsyncRead + AsyncWrite` wrapped in `capnp_rpc::twoparty::VatNetwork` |
| Per-call `requestTimeout` | `tokio::time::timeout(duration, rpc_call)` |
| `uuid.UUID` wire format | `uuid::Uuid` with `as_bytes()` for schema-compatible serialization |
| `(*CloudflaredClient).Close()` | `Drop` impl or explicit `shutdown()` cancelling the `VatNetwork` |

### [quic/cloudflared_server.go](../../../atoms/tunnelrpc/quic/cloudflared_server.md)

| Go construct | Rust equivalent |
| --- | --- |
| `Serve()` reads RPC messages, dispatches to `handleRPC()` | Implement Cap'n Proto server trait from `capnpc-rust`; each method → `async fn` |
| `HandleRequestFunc` function-typed callback | `Box<dyn Fn(ConnectRequest) -> BoxFuture<ConnectResponse> + Send + Sync>` |
| `select` with timeout on RPC handling | `tokio::time::timeout(duration, handle_rpc(msg)).await` |
| `switch` on RPC type in `handleRPC()` | `match` on which-field enum |

### [quic/session_client.go](../../../atoms/tunnelrpc/quic/session_client.md)

| Go construct | Rust equivalent |
| --- | --- |
| Same `capnp-rpc` transport pattern as `cloudflared_client` | Reuse common `RpcTransport` struct |
| `RegisterUdpSession` / `UnregisterUdpSession` | Generated async methods from `.capnp` schema |
| Per-call timeout | `tokio::time::timeout` wrapping each RPC future |

### [quic/session_server.go](../../../atoms/tunnelrpc/quic/session_server.md)

| Go construct | Rust equivalent |
| --- | --- |
| `SessionManagerServer.Serve()` dispatches with timeout | Cap'n Proto session manager server trait; `async fn` + `tokio::time::timeout` |
| `select` on context + RPC completion | `tokio::select!` with cancellation token and timeout |
| `switch` on Cap'n Proto which-field | `match` on generated enum; exhaust all variants |

### [registration_client.go](../../../atoms/tunnelrpc/registration_client.md)

| Go construct | Rust equivalent |
| --- | --- |
| `zombiezen.com/go/capnproto2/rpc` | `capnp-rpc` with `twoparty::VatNetwork` |
| `io.ReadWriteCloser` transport | `tokio::io::AsyncRead + AsyncWrite` |
| Per-call `requestTimeout` for `RegisterConnection`, `SendLocalConfiguration`, `GracefulShutdown` | `tokio::time::timeout(duration, rpc_future)` |
| Graceful shutdown: unregister then wait for grace period | `tokio::time::timeout(grace_period, unregister_rpc)` + `CancellationToken` |
| `RegistrationClient` Go interface | Rust trait with async methods; inject via `Arc<dyn RegistrationClient + Send + Sync>` |

### [registration_server.go](../../../atoms/tunnelrpc/registration_server.md)

| Go construct | Rust equivalent |
| --- | --- |
| `RegistrationServer.Serve()` | Cap'n Proto `registration_server::Server` trait with `async fn` methods |
| `select` for shutdown | `tokio::select!` with `CancellationToken` |
| `io.ReadWriteCloser` for RPC stream | `capnp_rpc::RpcSystem` over `AsyncRead + AsyncWrite + Unpin` |
| Zero if-branches (pure dispatch) | Rust port should remain equally thin |

### [utils.go](../../../atoms/tunnelrpc/utils.md)

| Go construct | Rust equivalent |
| --- | --- |
| `SafeTransport()` wraps stream with temporary-error retry | Middleware retrying on `io::ErrorKind::WouldBlock` / `Interrupted`; or rely on `tokio::io` internals |
| `isTemporaryError()` errno-level check | `match err.kind() { WouldBlock \| Interrupted => true, _ => false }` |
| `noopCapnpLogger` | Not needed; `capnp-rpc` uses `log`/`tracing` filterable at subscriber level |
| `NewClientConn()` / `NewServerConn()` | `capnp_rpc::new_client()` / `capnp_rpc::new_server()` |

## tunnelstate

### [conntracker.go](../../../atoms/tunnelstate/conntracker.md)

| Go construct | Rust equivalent |
| --- | --- |
| `sync.Mutex` protecting connection status map | `Arc<Mutex<HashMap<ConnIndex, ConnStatus>>>` or `DashMap` |
| `connection.Event` interface-based update | `match event { Connected(…) => …, Disconnected(…) => … }` |

## websocket

### [connection.go](../../../atoms/websocket/connection.md)

| Go construct | Rust equivalent |
| --- | --- |
| Dual WebSocket libraries (gorilla + gobwas) | Unify on `tokio_tungstenite` (single library) |
| Background goroutine sending periodic pings | `tokio::spawn` with `tokio::select!` on `interval.tick()` + `CancellationToken` |
| `sync.Mutex` protecting concurrent writes | `tokio::sync::Mutex<SplitSink<…>>` or channel-based write serialization |

## Crate Frequency Summary

The following Rust crates appear most frequently across atoms, forming the core
dependency set for the port:

| Rust crate | Occurrences | Replaces Go package(s) |
| --- | ---: | --- |
| `tokio` (runtime, sync, time, io, select) | 79 | goroutines, channels, context, timers |
| `tokio_util::sync::CancellationToken` | 25+ | `context.WithCancel`, `<-chan struct{}` |
| `quinn` | 12+ | `quic-go` |
| `capnp-rpc` / `capnpc-rust` | 12+ | `zombiezen.com/go/capnproto2` |
| `rustls` / `tokio_rustls` | 10+ | `crypto/tls` |
| `serde` / `serde_json` / `serde_yaml` | 10+ | `encoding/json`, `gopkg.in/yaml.v3` |
| `thiserror` | 8+ | `errors.New`, `fmt.Errorf`, `pkg/errors` |
| `clap` | 6+ | `urfave/cli/v2` |
| `reqwest` / `hyper` | 5+ | `net/http`, `golang.org/x/net/http2` |
| `tracing` / `tracing_subscriber` | 5+ | `zerolog`, `net/http/pprof` |
| `opentelemetry_sdk` / `opentelemetry_otlp` | 4+ | `go.opentelemetry.io/otel` |
| `DashMap` | 4+ | `sync.Map` |
| `uuid` | 3+ | `github.com/google/uuid` |
| `pnet_packet` / `etherparse` | 3+ | `gopacket`, `x/net/icmp` |
| `arc_swap` | 2+ | `sync.RWMutex` hot-read patterns |
