# S1/RAW — Go Source Behavior Baseline

- Baseline date: 20260320
- Scope: [cloudflare/cloudflared/tree/2026.3.0](https://github.com/cloudflare/cloudflared/tree/2026.3.0) Go source behavior extraction
- Coverage: 241 atoms across 30 catalogs (22 domain + 8 cross-cutting)
- Exit gate: **PASSED** — 100% atom coverage, all depth floors met, audit analysis coherent

## Analysis

- [audit-analysis](audit-analysis.md) — seven-dimension coverage analysis
- [atom-graphs](atom-graphs.md) — behavior-group flow diagrams

## Domain Catalogs

- [access-policies](catalogs/domain/access-policies.md)
- [capnp-rpc](catalogs/domain/capnp-rpc.md)
- [cli](catalogs/domain/cli.md)
- [config](catalogs/domain/config.md)
- [const-and-env](catalogs/domain/const-and-env.md)
- [crypto](catalogs/domain/crypto.md)
- [deployments](catalogs/domain/deployments/README.md)
- [edge-interactions](catalogs/domain/edge-interactions.md)
- [host-interactions](catalogs/domain/host-interactions.md)
- [ingress](catalogs/domain/ingress.md)
- [metrics](catalogs/domain/metrics.md)
- [observabilities](catalogs/domain/observabilities.md)
- [overwatch](catalogs/domain/overwatch.md)
- [platforms](catalogs/domain/platforms.md)
- [proxying](catalogs/domain/proxying.md)
- [sessions](catalogs/domain/sessions.md)
- [shared-state](catalogs/domain/shared-state.md)
- [state-machines](catalogs/domain/state-machines.md)
- [supervisor](catalogs/domain/supervisor.md)
- [tunnels](catalogs/domain/tunnels.md)
- [tunnels-transport](catalogs/domain/tunnels-transport.md)
- [upstream-api-contracts](catalogs/domain/upstream-api-contracts.md)

## Cross-Cutting Catalogs

Analytical lenses across domain catalogs; all referenced atoms are
already covered above.

- [concurrency](catalogs/cross-cutting/concurrency/README.md)
- [error-propagation](catalogs/cross-cutting/error-propagation/README.md)
- [features](catalogs/cross-cutting/features/README.md)
- [init-teardown](catalogs/cross-cutting/init-teardown/README.md)
- [platform-substrates](catalogs/cross-cutting/platform-substrates.md)
- [porting-friction](catalogs/cross-cutting/porting-friction/README.md)
- [tests](catalogs/cross-cutting/tests.md)
  - [tests-transport](catalogs/cross-cutting/tests-transport.md)
  - [tests-proxy-ingress](catalogs/cross-cutting/tests-proxy-ingress.md)
  - [tests-sessions-packets](catalogs/cross-cutting/tests-sessions-packets.md)
  - [tests-infrastructure](catalogs/cross-cutting/tests-infrastructure.md)
- [wire-protocol](catalogs/cross-cutting/wire-protocol/README.md)

## Atom Corpus

- [atom-context-map](atom-context-map.md) — scope guardrails and context map
- [atom-index](atom-index.md) — navigation index by Go package
- [atoms](atoms) — one behavior atom per non-vendor, non-test Go source file
