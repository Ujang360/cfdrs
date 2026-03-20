# Atom Behavior Atom Index

Generated from non-vendor, non-test Go implementation files using the Cloudflared permalink baseline at [cloudflare/cloudflared/tree/2026.3.0](https://github.com/cloudflare/cloudflared/tree/2026.3.0).

- Atom count: 241
- Module count: 41

## Modules

- carrier: 2 atoms
- cfapi: 9 atoms
- cfio: 1 atoms
- client: 1 atoms
- cmd: 43 atoms
- config: 3 atoms
- connection: 15 atoms
- credentials: 2 atoms
- datagramsession: 4 atoms
- diagnostic: 20 atoms
- edgediscovery: 8 atoms
- features: 2 atoms
- fips: 2 atoms
- flow: 2 atoms
- hello: 1 atoms
- ingress: 19 atoms
- internal: 1 atoms
- ipaccess: 1 atoms
- logger: 3 atoms
- management: 6 atoms
- metrics: 3 atoms
- mocks: 2 atoms
- orchestration: 3 atoms
- overwatch: 2 atoms
- packet: 5 atoms
- proxy: 3 atoms
- quic: 17 atoms
- retry: 1 atoms
- signal: 1 atoms
- socks: 6 atoms
- sshgen: 1 atoms
- stream: 2 atoms
- supervisor: 8 atoms
- tlsconfig: 4 atoms
- token: 9 atoms
- tracing: 3 atoms
- tunnelrpc: 20 atoms
- tunnelstate: 1 atoms
- validation: 1 atoms
- watcher: 2 atoms
- websocket: 2 atoms

## Atom Links by Module

### carrier

- [carrier/carrier.go](atoms/carrier/carrier.md)
- [carrier/websocket.go](atoms/carrier/websocket.md)

### cfapi

- [cfapi/base_client.go](atoms/cfapi/base_client.md)
- [cfapi/client.go](atoms/cfapi/client.md)
- [cfapi/hostname.go](atoms/cfapi/hostname.md)
- [cfapi/ip_route.go](atoms/cfapi/ip_route.md)
- [cfapi/ip_route_filter.go](atoms/cfapi/ip_route_filter.md)
- [cfapi/tunnel.go](atoms/cfapi/tunnel.md)
- [cfapi/tunnel_filter.go](atoms/cfapi/tunnel_filter.md)
- [cfapi/virtual_network.go](atoms/cfapi/virtual_network.md)
- [cfapi/virtual_network_filter.go](atoms/cfapi/virtual_network_filter.md)

### cfio

- [cfio/copy.go](atoms/cfio/copy.md)

### client

- [client/config.go](atoms/client/config.md)

### cmd

- [cmd/cloudflared/access/carrier.go](atoms/cmd/cloudflared/access/carrier.md)
- [cmd/cloudflared/access/cmd.go](atoms/cmd/cloudflared/access/cmd.md)
- [cmd/cloudflared/access/validation.go](atoms/cmd/cloudflared/access/validation.md)
- [cmd/cloudflared/app_forward_service.go](atoms/cmd/cloudflared/app_forward_service.md)
- [cmd/cloudflared/app_service.go](atoms/cmd/cloudflared/app_service.md)
- [cmd/cloudflared/cliutil/build_info.go](atoms/cmd/cloudflared/cliutil/build_info.md)
- [cmd/cloudflared/cliutil/deprecated.go](atoms/cmd/cloudflared/cliutil/deprecated.md)
- [cmd/cloudflared/cliutil/errors.go](atoms/cmd/cloudflared/cliutil/errors.md)
- [cmd/cloudflared/cliutil/handler.go](atoms/cmd/cloudflared/cliutil/handler.md)
- [cmd/cloudflared/cliutil/logger.go](atoms/cmd/cloudflared/cliutil/logger.md)
- [cmd/cloudflared/cliutil/management.go](atoms/cmd/cloudflared/cliutil/management.md)
- [cmd/cloudflared/common_service.go](atoms/cmd/cloudflared/common_service.md)
- [cmd/cloudflared/flags/flags.go](atoms/cmd/cloudflared/flags/flags.md)
- [cmd/cloudflared/generic_service.go](atoms/cmd/cloudflared/generic_service.md)
- [cmd/cloudflared/linux_service.go](atoms/cmd/cloudflared/linux_service.md)
- [cmd/cloudflared/macos_service.go](atoms/cmd/cloudflared/macos_service.md)
- [cmd/cloudflared/main.go](atoms/cmd/cloudflared/main.md)
- [cmd/cloudflared/management/cmd.go](atoms/cmd/cloudflared/management/cmd.md)
- [cmd/cloudflared/proxydns/cmd.go](atoms/cmd/cloudflared/proxydns/cmd.md)
- [cmd/cloudflared/service_template.go](atoms/cmd/cloudflared/service_template.md)
- [cmd/cloudflared/tail/cmd.go](atoms/cmd/cloudflared/tail/cmd.md)
- [cmd/cloudflared/tunnel/cmd.go](atoms/cmd/cloudflared/tunnel/cmd.md)
- [cmd/cloudflared/tunnel/configuration.go](atoms/cmd/cloudflared/tunnel/configuration.md)
- [cmd/cloudflared/tunnel/credential_finder.go](atoms/cmd/cloudflared/tunnel/credential_finder.md)
- [cmd/cloudflared/tunnel/filesystem.go](atoms/cmd/cloudflared/tunnel/filesystem.md)
- [cmd/cloudflared/tunnel/info.go](atoms/cmd/cloudflared/tunnel/info.md)
- [cmd/cloudflared/tunnel/ingress_subcommands.go](atoms/cmd/cloudflared/tunnel/ingress_subcommands.md)
- [cmd/cloudflared/tunnel/login.go](atoms/cmd/cloudflared/tunnel/login.md)
- [cmd/cloudflared/tunnel/quick_tunnel.go](atoms/cmd/cloudflared/tunnel/quick_tunnel.md)
- [cmd/cloudflared/tunnel/signal.go](atoms/cmd/cloudflared/tunnel/signal.md)
- [cmd/cloudflared/tunnel/subcommand_context.go](atoms/cmd/cloudflared/tunnel/subcommand_context.md)
- [cmd/cloudflared/tunnel/subcommand_context_teamnet.go](atoms/cmd/cloudflared/tunnel/subcommand_context_teamnet.md)
- [cmd/cloudflared/tunnel/subcommand_context_vnets.go](atoms/cmd/cloudflared/tunnel/subcommand_context_vnets.md)
- [cmd/cloudflared/tunnel/subcommands.go](atoms/cmd/cloudflared/tunnel/subcommands.md)
- [cmd/cloudflared/tunnel/tag.go](atoms/cmd/cloudflared/tunnel/tag.md)
- [cmd/cloudflared/tunnel/teamnet_subcommands.go](atoms/cmd/cloudflared/tunnel/teamnet_subcommands.md)
- [cmd/cloudflared/tunnel/vnets_subcommands.go](atoms/cmd/cloudflared/tunnel/vnets_subcommands.md)
- [cmd/cloudflared/updater/check.go](atoms/cmd/cloudflared/updater/check.md)
- [cmd/cloudflared/updater/service.go](atoms/cmd/cloudflared/updater/service.md)
- [cmd/cloudflared/updater/update.go](atoms/cmd/cloudflared/updater/update.md)
- [cmd/cloudflared/updater/workers_service.go](atoms/cmd/cloudflared/updater/workers_service.md)
- [cmd/cloudflared/updater/workers_update.go](atoms/cmd/cloudflared/updater/workers_update.md)
- [cmd/cloudflared/windows_service.go](atoms/cmd/cloudflared/windows_service.md)

### config

- [config/configuration.go](atoms/config/configuration.md)
- [config/manager.go](atoms/config/manager.md)
- [config/model.go](atoms/config/model.md)

### connection

- [connection/connection.go](atoms/connection/connection.md)
- [connection/control.go](atoms/connection/control.md)
- [connection/errors.go](atoms/connection/errors.md)
- [connection/event.go](atoms/connection/event.md)
- [connection/header.go](atoms/connection/header.md)
- [connection/http2.go](atoms/connection/http2.md)
- [connection/json.go](atoms/connection/json.md)
- [connection/metrics.go](atoms/connection/metrics.md)
- [connection/observer.go](atoms/connection/observer.md)
- [connection/protocol.go](atoms/connection/protocol.md)
- [connection/quic.go](atoms/connection/quic.md)
- [connection/quic_connection.go](atoms/connection/quic_connection.md)
- [connection/quic_datagram_v2.go](atoms/connection/quic_datagram_v2.md)
- [connection/quic_datagram_v3.go](atoms/connection/quic_datagram_v3.md)
- [connection/tunnelsforha.go](atoms/connection/tunnelsforha.md)

### credentials

- [credentials/credentials.go](atoms/credentials/credentials.md)
- [credentials/origin_cert.go](atoms/credentials/origin_cert.md)

### datagramsession

- [datagramsession/event.go](atoms/datagramsession/event.md)
- [datagramsession/manager.go](atoms/datagramsession/manager.md)
- [datagramsession/metrics.go](atoms/datagramsession/metrics.md)
- [datagramsession/session.go](atoms/datagramsession/session.md)

### diagnostic

- [diagnostic/client.go](atoms/diagnostic/client.md)
- [diagnostic/consts.go](atoms/diagnostic/consts.md)
- [diagnostic/diagnostic.go](atoms/diagnostic/diagnostic.md)
- [diagnostic/diagnostic_utils.go](atoms/diagnostic/diagnostic_utils.md)
- [diagnostic/error.go](atoms/diagnostic/error.md)
- [diagnostic/handlers.go](atoms/diagnostic/handlers.md)
- [diagnostic/log_collector.go](atoms/diagnostic/log_collector.md)
- [diagnostic/log_collector_docker.go](atoms/diagnostic/log_collector_docker.md)
- [diagnostic/log_collector_host.go](atoms/diagnostic/log_collector_host.md)
- [diagnostic/log_collector_kubernetes.go](atoms/diagnostic/log_collector_kubernetes.md)
- [diagnostic/log_collector_utils.go](atoms/diagnostic/log_collector_utils.md)
- [diagnostic/network/collector.go](atoms/diagnostic/network/collector.md)
- [diagnostic/network/collector_unix.go](atoms/diagnostic/network/collector_unix.md)
- [diagnostic/network/collector_utils.go](atoms/diagnostic/network/collector_utils.md)
- [diagnostic/network/collector_windows.go](atoms/diagnostic/network/collector_windows.md)
- [diagnostic/system_collector.go](atoms/diagnostic/system_collector.md)
- [diagnostic/system_collector_linux.go](atoms/diagnostic/system_collector_linux.md)
- [diagnostic/system_collector_macos.go](atoms/diagnostic/system_collector_macos.md)
- [diagnostic/system_collector_utils.go](atoms/diagnostic/system_collector_utils.md)
- [diagnostic/system_collector_windows.go](atoms/diagnostic/system_collector_windows.md)

### edgediscovery

- [edgediscovery/allregions/address.go](atoms/edgediscovery/allregions/address.md)
- [edgediscovery/allregions/discovery.go](atoms/edgediscovery/allregions/discovery.md)
- [edgediscovery/allregions/region.go](atoms/edgediscovery/allregions/region.md)
- [edgediscovery/allregions/regions.go](atoms/edgediscovery/allregions/regions.md)
- [edgediscovery/allregions/usedby.go](atoms/edgediscovery/allregions/usedby.md)
- [edgediscovery/dial.go](atoms/edgediscovery/dial.md)
- [edgediscovery/edgediscovery.go](atoms/edgediscovery/edgediscovery.md)
- [edgediscovery/protocol.go](atoms/edgediscovery/protocol.md)

### features

- [features/features.go](atoms/features/features.md)
- [features/selector.go](atoms/features/selector.md)

### fips

- [fips/fips.go](atoms/fips/fips.md)
- [fips/nofips.go](atoms/fips/nofips.md)

### flow

- [flow/limiter.go](atoms/flow/limiter.md)
- [flow/metrics.go](atoms/flow/metrics.md)

### hello

- [hello/hello.go](atoms/hello/hello.md)

### ingress

- [ingress/config.go](atoms/ingress/config.md)
- [ingress/icmp_darwin.go](atoms/ingress/icmp_darwin.md)
- [ingress/icmp_generic.go](atoms/ingress/icmp_generic.md)
- [ingress/icmp_linux.go](atoms/ingress/icmp_linux.md)
- [ingress/icmp_metrics.go](atoms/ingress/icmp_metrics.md)
- [ingress/icmp_posix.go](atoms/ingress/icmp_posix.md)
- [ingress/icmp_windows.go](atoms/ingress/icmp_windows.md)
- [ingress/ingress.go](atoms/ingress/ingress.md)
- [ingress/middleware/jwtvalidator.go](atoms/ingress/middleware/jwtvalidator.md)
- [ingress/middleware/middleware.go](atoms/ingress/middleware/middleware.md)
- [ingress/origin_connection.go](atoms/ingress/origin_connection.md)
- [ingress/origin_dialer.go](atoms/ingress/origin_dialer.md)
- [ingress/origin_icmp_proxy.go](atoms/ingress/origin_icmp_proxy.md)
- [ingress/origin_proxy.go](atoms/ingress/origin_proxy.md)
- [ingress/origin_service.go](atoms/ingress/origin_service.md)
- [ingress/origins/dns.go](atoms/ingress/origins/dns.md)
- [ingress/origins/metrics.go](atoms/ingress/origins/metrics.md)
- [ingress/packet_router.go](atoms/ingress/packet_router.md)
- [ingress/rule.go](atoms/ingress/rule.md)

### internal

- [internal/test/wstest.go](atoms/internal/test/wstest.md)

### ipaccess

- [ipaccess/access.go](atoms/ipaccess/access.md)

### logger

- [logger/configuration.go](atoms/logger/configuration.md)
- [logger/console.go](atoms/logger/console.md)
- [logger/create.go](atoms/logger/create.md)

### management

- [management/events.go](atoms/management/events.md)
- [management/logger.go](atoms/management/logger.md)
- [management/middleware.go](atoms/management/middleware.md)
- [management/service.go](atoms/management/service.md)
- [management/session.go](atoms/management/session.md)
- [management/token.go](atoms/management/token.md)

### metrics

- [metrics/config.go](atoms/metrics/config.md)
- [metrics/metrics.go](atoms/metrics/metrics.md)
- [metrics/readiness.go](atoms/metrics/readiness.md)

### mocks

- [mocks/mock_limiter.go](atoms/mocks/mock_limiter.md)
- [mocks/mockgen.go](atoms/mocks/mockgen.md)

### orchestration

- [orchestration/config.go](atoms/orchestration/config.md)
- [orchestration/metrics.go](atoms/orchestration/metrics.md)
- [orchestration/orchestrator.go](atoms/orchestration/orchestrator.md)

### overwatch

- [overwatch/app_manager.go](atoms/overwatch/app_manager.md)
- [overwatch/manager.go](atoms/overwatch/manager.md)

### packet

- [packet/decoder.go](atoms/packet/decoder.md)
- [packet/encoder.go](atoms/packet/encoder.md)
- [packet/funnel.go](atoms/packet/funnel.md)
- [packet/packet.go](atoms/packet/packet.md)
- [packet/session.go](atoms/packet/session.md)

### proxy

- [proxy/logger.go](atoms/proxy/logger.md)
- [proxy/metrics.go](atoms/proxy/metrics.md)
- [proxy/proxy.go](atoms/proxy/proxy.md)

### quic

- [quic/constants.go](atoms/quic/constants.md)
- [quic/conversion.go](atoms/quic/conversion.md)
- [quic/datagram.go](atoms/quic/datagram.md)
- [quic/datagramv2.go](atoms/quic/datagramv2.md)
- [quic/metrics.go](atoms/quic/metrics.md)
- [quic/param_unix.go](atoms/quic/param_unix.md)
- [quic/param_windows.go](atoms/quic/param_windows.md)
- [quic/safe_stream.go](atoms/quic/safe_stream.md)
- [quic/tracing.go](atoms/quic/tracing.md)
- [quic/v3/datagram.go](atoms/quic/v3/datagram.md)
- [quic/v3/datagram_errors.go](atoms/quic/v3/datagram_errors.md)
- [quic/v3/icmp.go](atoms/quic/v3/icmp.md)
- [quic/v3/manager.go](atoms/quic/v3/manager.md)
- [quic/v3/metrics.go](atoms/quic/v3/metrics.md)
- [quic/v3/muxer.go](atoms/quic/v3/muxer.md)
- [quic/v3/request.go](atoms/quic/v3/request.md)
- [quic/v3/session.go](atoms/quic/v3/session.md)

### retry

- [retry/backoffhandler.go](atoms/retry/backoffhandler.md)

### signal

- [signal/safe_signal.go](atoms/signal/safe_signal.md)

### socks

- [socks/auth_handler.go](atoms/socks/auth_handler.md)
- [socks/authenticator.go](atoms/socks/authenticator.md)
- [socks/connection_handler.go](atoms/socks/connection_handler.md)
- [socks/dialer.go](atoms/socks/dialer.md)
- [socks/request.go](atoms/socks/request.md)
- [socks/request_handler.go](atoms/socks/request_handler.md)

### sshgen

- [sshgen/sshgen.go](atoms/sshgen/sshgen.md)

### stream

- [stream/debug.go](atoms/stream/debug.md)
- [stream/stream.go](atoms/stream/stream.md)

### supervisor

- [supervisor/conn_aware_logger.go](atoms/supervisor/conn_aware_logger.md)
- [supervisor/external_control.go](atoms/supervisor/external_control.md)
- [supervisor/fuse.go](atoms/supervisor/fuse.md)
- [supervisor/metrics.go](atoms/supervisor/metrics.md)
- [supervisor/pqtunnels.go](atoms/supervisor/pqtunnels.md)
- [supervisor/supervisor.go](atoms/supervisor/supervisor.md)
- [supervisor/tunnel.go](atoms/supervisor/tunnel.md)
- [supervisor/tunnelsforha.go](atoms/supervisor/tunnelsforha.md)

### tlsconfig

- [tlsconfig/certreloader.go](atoms/tlsconfig/certreloader.md)
- [tlsconfig/cloudflare_ca.go](atoms/tlsconfig/cloudflare_ca.md)
- [tlsconfig/hello_ca.go](atoms/tlsconfig/hello_ca.md)
- [tlsconfig/tlsconfig.go](atoms/tlsconfig/tlsconfig.md)

### token

- [token/encrypt.go](atoms/token/encrypt.md)
- [token/launch_browser_darwin.go](atoms/token/launch_browser_darwin.md)
- [token/launch_browser_other.go](atoms/token/launch_browser_other.md)
- [token/launch_browser_unix.go](atoms/token/launch_browser_unix.md)
- [token/launch_browser_windows.go](atoms/token/launch_browser_windows.md)
- [token/path.go](atoms/token/path.md)
- [token/shell.go](atoms/token/shell.md)
- [token/token.go](atoms/token/token.md)
- [token/transfer.go](atoms/token/transfer.md)

### tracing

- [tracing/client.go](atoms/tracing/client.md)
- [tracing/identity.go](atoms/tracing/identity.md)
- [tracing/tracing.go](atoms/tracing/tracing.md)

### tunnelrpc

- [tunnelrpc/metrics/metrics.go](atoms/tunnelrpc/metrics/metrics.md)
- [tunnelrpc/pogs/cloudflared_server.go](atoms/tunnelrpc/pogs/cloudflared_server.md)
- [tunnelrpc/pogs/configuration_manager.go](atoms/tunnelrpc/pogs/configuration_manager.md)
- [tunnelrpc/pogs/errors.go](atoms/tunnelrpc/pogs/errors.md)
- [tunnelrpc/pogs/quic_metadata_protocol.go](atoms/tunnelrpc/pogs/quic_metadata_protocol.md)
- [tunnelrpc/pogs/registration_server.go](atoms/tunnelrpc/pogs/registration_server.md)
- [tunnelrpc/pogs/session_manager.go](atoms/tunnelrpc/pogs/session_manager.md)
- [tunnelrpc/pogs/tag.go](atoms/tunnelrpc/pogs/tag.md)
- [tunnelrpc/proto/quic_metadata_protocol.capnp.go](atoms/tunnelrpc/proto/quic_metadata_protocol.capnp)
- [tunnelrpc/proto/tunnelrpc.capnp.go](atoms/tunnelrpc/proto/tunnelrpc.capnp)
- [tunnelrpc/quic/cloudflared_client.go](atoms/tunnelrpc/quic/cloudflared_client.md)
- [tunnelrpc/quic/cloudflared_server.go](atoms/tunnelrpc/quic/cloudflared_server.md)
- [tunnelrpc/quic/protocol.go](atoms/tunnelrpc/quic/protocol.md)
- [tunnelrpc/quic/request_client_stream.go](atoms/tunnelrpc/quic/request_client_stream.md)
- [tunnelrpc/quic/request_server_stream.go](atoms/tunnelrpc/quic/request_server_stream.md)
- [tunnelrpc/quic/session_client.go](atoms/tunnelrpc/quic/session_client.md)
- [tunnelrpc/quic/session_server.go](atoms/tunnelrpc/quic/session_server.md)
- [tunnelrpc/registration_client.go](atoms/tunnelrpc/registration_client.md)
- [tunnelrpc/registration_server.go](atoms/tunnelrpc/registration_server.md)
- [tunnelrpc/utils.go](atoms/tunnelrpc/utils.md)

### tunnelstate

- [tunnelstate/conntracker.go](atoms/tunnelstate/conntracker.md)

### validation

- [validation/validation.go](atoms/validation/validation.md)

### watcher

- [watcher/file.go](atoms/watcher/file.md)
- [watcher/notify.go](atoms/watcher/notify.md)

### websocket

- [websocket/connection.go](atoms/websocket/connection.md)
- [websocket/websocket.go](atoms/websocket/websocket.md)
