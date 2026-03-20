# RR — Rust Rewrite

Phase 1 of the cfdrs program: rewriting
[cloudflare/cloudflared](https://github.com/cloudflare/cloudflared) in Rust.

**Goal:** Produce a behaviorally equivalent Rust binary for the scoped
feature set, verified by oracle-driven parity testing.

**Scope boundary:** cloudflare/cloudflared @ tag `2026.3.0`. Deliberately
scoped — what RR leaves behind becomes FC's starting point.

## Strata

| Stratum | Tag | Purpose | Status |
|---------|-----|---------|--------|
| 1 | [RAW](s1/README.md) | Go source behavior extraction | Closed |
| 2 | SUBSTRATE | Rust-aware design decisions | Next |
| 3 | COOK | Rust implementation | Not started |
| 4 | SERVING | Verification, conformance, deploy-ready | Not started |

## S1/RAW — Raw Material Gathering

Language-neutral, full-fidelity behavioral documentation of the Go
implementation. Context-agnostic — can be re-perspectivized into any
target language.

**Location:** [s1](s1/README.md)

| Phase | Description | Artifact | Status |
|---|---|---|---|
| 1.1 | Go AST audit — behavioral atomization | `atoms/` — 241 atoms across 41 modules | Done |
| 1.2 | Domain cataloging | `catalogs/domain/` — 22 catalogs | Done |
| 1.3 | Cross-cutting cataloging | `catalogs/cross-cutting/` — 8 catalogs + sub-files | Done |
| 1.4 | Tests oracle — behavioral contracts from Go test suite | `catalogs/cross-cutting/tests` — ~280 contracts | Done |
| 1.5 | Oracle binary pin — Go binary at 2026.3.0, hash-pinned | `oracle/go-2026.3.0-binary.md` | Done (human task) |
| 1.6 | Audit analysis — Jaccard, Gini, hub atoms, clusters | [audit-analysis.md](s1/audit-analysis.md) | Done |

| Metric | Value |
|---|---|
| Atoms | 241 (100% coverage) |
| Catalogs | 30 (22 domain + 8 cross-cutting) |
| Gini coefficient | 0.3464 (below 0.40 alarm) |
| Depth floor | 45.0 minimum across all catalogs |
| Hub atoms (membership ≥8) | 47 |
| Hub atoms (membership ≥10) | 20 — critical path |
| Catalog clusters | 8 natural clusters from Jaccard analysis |

**Exit gate: PASSED** — 100% atom coverage, all depth floors met, audit
analysis coherent, oracle binary pinnable.

## S2/SUBSTRATE — Raw Materials Prepped for Cooking

Transform raw stratum into Rust-aware design decisions. Every artifact
here is language-aware, paradigm-aware, ecosystem-aware. This is where
Go behavior becomes Rust architecture.

**Location:** `s2/`

**Methodology:** Two-model handoff on all prompts — Sonnet 4.6 (first
pass) + GPT 5.4 (verification pass).

| Phase | Description | Artifact |
|---|---|---|
| 2.1 | Dependencies vetting — plumbing + behavioral layers | `s2/dependencies.md` |
| 2.2 | Scoping — platform matrix, MoSCoW, explicit non-ports | `s2/scope.md` |
| 2.3 | Risk register — probability × impact from catalogs | `s2/risks.md` |
| 2.4 | ADRs — one per decision, references catalog evidence | `s2/adr/NNN-*.md` |
| 2.5 | Invariants — behavioral invariants → proptest properties | `s2/invariants.md` |
| 2.6 | Architecture — crate boundaries from Jaccard clusters | `s2/architecture.md` |
| 2.7 | Parity harness — TOML contracts, oracle, red-by-default | `s2/parity-design.md` |
| 2.8 | Stages plan — phased from critical path + dep graph | `s2/stages/phase-*.md` |
| 2.9 | Environment + guardrails — workspace, style, CI/CD | `s2/environment.md` |
| 2.10 | Coherency audit — full traceability chain | `s2/coherency-report.md` |

### Dependency vetting layers (Phase 2.1)

**Layer 1 — Plumbing:** Low Jaccard overlap, independent cluster, low-risk.

```text
tokio, tokio-util, bytes, pin-project
rustls / boring (FIPS-aware TLS)
serde, serde_json, serde_yaml
tracing, tracing-subscriber
prometheus client
clap
anyhow / thiserror
```

**Layer 2 — Behavioral:** High Jaccard overlap, tunnel core + control
plane clusters, high-stakes.

```text
quinn / tokio-quiche       ← QUIC (the critical decision)
capnp                      ← Cap'n Proto RPC
boring (BoringSSL)         ← PQ crypto surface
hyper / h2                 ← HTTP2
ractor / tokio channels    ← actor/supervision
```

### Critical path (from hub atom analysis)

Atoms at membership ≥12 must have Rust type interfaces designed before
any other crate builds against them:

```text
tunnel/configuration        (14 catalogs) — central dispatch
connection/control          (14 catalogs) — RPC control stream backbone
tunnel/cmd                  (13 catalogs) — primary tunnel CLI command
connection/protocol         (13 catalogs) — protocol selection
management/service          (12 catalogs) — management runtime
quic/v3/session             (12 catalogs) — QUIC session lifecycle
supervisor/tunnel           (12 catalogs) — supervisor loop
```

**Exit gate:** Features traceable to scope. Scope traceable to ADRs.
ADRs traceable to crates. Crates traceable to stages. Parity harness
covers all scoped features. No orphaned decisions. Full coherency chain.

## S3/COOK — Implementation

Cognitively-bounded implementation. Each stage is precisely scoped to a
context window / cognitive load unit. Red parity tests turn green
progressively.

**Location:** `s3/` + codebase

**Methodology:** Two-model handoff — GPT 5.3 Codex (first pass) +
Opus 4.6 (verification). One crate per session. Parity TOML written
before implementation.

| Stage | Scope | Entry condition |
|---|---|---|
| 3.0 | Foundation — shared types, error taxonomy, trait interfaces | S2 exit gate |
| 3.1 | Independent crates — cli, config, const-and-env, crypto, overwatch, metrics | 3.0 |
| 3.2 | Control plane — capnp-rpc, tunnelrpc, registration client | 3.0 |
| 3.3 | Transport — connection/http2, connection/quic, tunnels-transport | 3.0 + 3.2 |
| 3.4 | Session + datagram — datagramsession, quic/v3, sessions | 3.3 |
| 3.5 | Supervisor + orchestration | 3.3 + 3.4 |
| 3.6 | Ingress + proxy — ingress, proxy, carrier, socks | 3.3 + 3.5 |
| 3.7 | Management + access — management, access-policies, token | 3.5 + 3.6 |
| 3.8 | CLI + binary assembly | All prior |
| 3.9 | Platform + service variants — FIPS, systemd, launchd, Windows | 3.8 |

**Exit gate:** All parity TOMLs for every stage are green.

## S4/SERVING — Verification, Conformance, Deploy-Ready

Prove behavioral equivalence between Go oracle and Rust implementation.

**Location:** `s4/`

| Phase | Activity | Output |
|---|---|---|
| 4.1 | Parity execution — all TOMLs vs Rust binary | Green/red report |
| 4.2 | Behavioral diff — Go oracle vs Rust on identical inputs | Delta catalog |
| 4.3 | Edge case discovery | New TOML contracts |
| 4.4 | Fuzz equivalence — fuzz-driven oracle comparison | Divergence log |
| 4.5 | Performance baseline — Rust vs Go | Perf delta report |
| 4.6 | Platform verification | Platform sign-off |
| 4.7 | Sign-off | Release artifact |

**Sign-off criteria:** All parity TOMLs green. No P0/P1 behavioral
divergences unresolved. Performance within bounds. Full platform matrix.
All catalog risks resolved or accepted.

## Feedback Loops

```text
S4 → S3: Behavioral diff reveals parity gap → add TOML, re-implement
S3 → S2: Scope miscalibration → revise ADR, update stage plan
S2 → S1: Missing behavioral surface → patch catalog
Any → audit-analysis: New findings update hub atom rankings or clusters
```

All upward revisions are lightweight patches, not full re-runs. The
coherency audit (Phase 2.10) validates the chain after any revision.
