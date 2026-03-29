# RR — Rust Rewrite

Phase 1 of the [cfdrs](../../README.md) program: rewriting
[cloudflare/cloudflared](https://github.com/cloudflare/cloudflared) in Rust.

**Goal:** Produce a behaviorally equivalent Rust binary for the scoped
feature set, verified by oracle-driven parity testing.

**Scope boundary:** cloudflare/cloudflared @ tag `2026.3.0`. Deliberately
scoped — what RR leaves behind becomes FC's starting point.

## Strata

| Stratum | Tag | Purpose | Status |
| --------- | ----- | --------- | -------- |
| 1 | [RAW](s1/README.md) | Go source behavior extraction | Closed |
| 2 | [SUBSTRATE](#s2substrate--raw-materials-prepped-for-cooking) | Rust-aware design decisions | Active |
| 3 | COOK | Rust implementation | Not started |
| 4 | SERVING | Verification, conformance, deploy-ready | Not started |

## S1/RAW — Raw Material Gathering

Language-neutral, full-fidelity behavioral documentation of the Go
implementation. Context-agnostic — can be re-perspectivized into any
target language.

**Location:** [s1](s1/README.md)

| Phase | Description | Artifact | Status |
| --- | --- | --- | --- |
| 1.1 | Go AST audit — behavioral atomization | `atoms/` — 241 atoms across 41 modules | Done |
| 1.2 | Domain cataloging | `catalogs/domain/` — 22 catalogs | Done |
| 1.3 | Cross-cutting cataloging | `catalogs/cross-cutting/` — 8 catalogs + sub-files | Done |
| 1.4 | Tests oracle — behavioral contracts from Go test suite | `catalogs/cross-cutting/tests` — ~280 contracts | Done |
| 1.5 | Oracle binary pin — Go binary at 2026.3.0, hash-pinned | `oracle/go-2026.3.0-binary.md` | Done (human task) |
| 1.6 | Audit analysis — Jaccard, Gini, hub atoms, clusters | [audit-analysis.md](s1/audit-analysis.md) | Done |

| Metric | Value |
| --- | --- |
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

**Location:** [s2/](s2/)

**Methodology:** Two-model handoff on all prompts — Sonnet 4.6 (first
pass) + GPT 5.4 (verification pass).

| Phase | Description | Artifact | Status |
| --- | --- | --- | --- |
| 2.1 | Dependencies vetting — 9 capability layers, T/M/S scored | [shopping cart](s2/dependency-shopping-cart.md), [decisions](s2/dependency-decisions.md) | Closed |
| 2.2 | Scoping — platform matrix, MoSCoW, explicit non-ports | [scope](s2/scope.md) | Closed |
| 2.3 | Risk register — probability × impact from 6 catalog domains | [risks](s2/risks.md) | Closed |
| 2.4 | ADRs — architecture decisions, referenced to catalog evidence | [ADR index](s2/adr/README.md) | Closed |
| 2.5 | Invariants — behavioral invariants → proptest properties | [invariants](s2/invariants.md) | Closed |
| 2.6 | Architecture — crate boundaries from Jaccard clusters | [architecture](s2/architecture.md) | Closed |
| 2.7 | Parity harness — TOML contracts, oracle, red-by-default | [parity-design](s2/parity-design.md) | Closed |
| 2.8 | Stages plan — phased from critical path + dep graph | [stages](s2/stages/README.md) | Closed |
| 2.9 | Environment + guardrails — workspace, style, CI/CD | `s2/environment.md` | — |
| 2.10 | Coherency audit — full traceability chain | `s2/coherency-report.md` | — |

### Dependency capability layers (Phase 2.1)

The [dependency decisions](s2/dependency-decisions.md) document organizes
capabilities into 9 layers, each traced to S1 catalog evidence and scored on
trustworthy/maturity/security axes. Research and alternatives are in the
[shopping cart](s2/dependency-shopping-cart.md).

| Layer | Scope | Key capabilities |
| --- | --- | --- |
| L1 | Runtime and Concurrency | tokio, parking_lot, arc-swap, dashmap, mimalloc |
| L2 | Serialization and Data | serde, capnp 0.25.x, prost |
| L3 | Transport | tokio-quiche (primary), hyper 1.x, h2 0.4.x, tower, BoringSSL |
| L4 | Observability | tracing, prometheus-client, opentelemetry |
| L5 | Error Handling | thiserror — all library crates. anyhow — OUT entirely. |
| L6 | CLI and Configuration | clap derive, chrono, dirs |
| L7 | Domain-Specific | backon, ipnet, regex, axum, ractor, flate2/zip |
| L8 | Crypto and Security | RustCrypto suite, jsonwebtoken, crypto_box |
| L9 | Platform | nix, systemd (sd-notify), socket2 |

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

### Phase dependency flow

Each S2 phase consumes prior outputs and produces inputs for downstream
phases. The flow below governs phase sequencing:

```text
2.1 Decisions ──┬──→ 2.2 Scope ──┬──→ 2.3 Risks ──┬──→ 2.4 ADRs ──→ 2.5 Invariants
                │               │               │               │
                └───────────────┴───────────────┴───────────────┘
                                        ↓
                              2.6 Architecture
                                        ↓
                              2.7 Parity Harness
                                        ↓
                              2.8 Stages Plan
                                        ↓
                              2.9 Environment
                                        ↓
                              2.10 Coherency Audit
```

| Metric | Value |
| --- | --- |
| Dependency layers | 9 across [decisions](s2/dependency-decisions.md) |
| Scope classifications | 127 feature atoms (33 Must / 68 Should / 18 Could / 8 Won't) |
| Platform atoms | 31 ([scope](s2/scope.md)) |
| Risks | 31 across 6 domains ([risks](s2/risks.md)) |
| ADRs | 19 total — 14 decided, 1 open, 2 deferred, 2 resolved inline |
| Invariants | 29 (6 architectural + 23 domain) across 10 prefixes |

**Exit gate:** Features traceable to scope. Scope traceable to ADRs.
ADRs traceable to crates. Crates traceable to stages. Parity harness
covers all scoped features. No orphaned decisions. Full coherency chain
validated by phase 2.10.

## S3/COOK — Implementation

Cognitively-bounded implementation. Each stage is precisely scoped to a
context window / cognitive load unit. Red parity tests turn green
progressively.

**Location:** `s3/` + codebase

**Methodology:** Two-model handoff — GPT 5.3 Codex (first pass) +
Opus 4.6 (verification). One crate per session. Parity TOML written
before implementation.

| Stage | Scope | Crates | Must | Should | Skip | Fuzz | Total | Phase doc |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 3.0 | Foundation | 3 | 6 | 0 | 0 | 0 | 6 | [phase-3.0](s2/stages/phase-3.0.md) |
| 3.1 | Independent crates | 7 | 6 | 46 | 11 | 1 | 64 | [phase-3.1](s2/stages/phase-3.1.md) |
| 3.2 | Control plane | 1 | 10 | 0 | 0 | 0 | 10 | [phase-3.2](s2/stages/phase-3.2.md) |
| 3.3 | Transport | 2 | 30 | 8 | 6 | 0 | 44 | [phase-3.3](s2/stages/phase-3.3.md) |
| 3.4 | Sessions | 1 | 52 | 11 | 0 | 6 | 69 | [phase-3.4](s2/stages/phase-3.4.md) |
| 3.5 | Config runtime + supervisor | 2 | 1 | 19 | 0 | 0 | 20 | [phase-3.5](s2/stages/phase-3.5.md) |
| 3.6 | Ingress proxy | 1 | 37 | 26 | 0 | 1 | 64 | [phase-3.6](s2/stages/phase-3.6.md) |
| 3.7 | Management + diagnostics | 2 | 6 | 32 | 1 | 0 | 39 | [phase-3.7](s2/stages/phase-3.7.md) |
| 3.8 | Application + CLI | 5 | 9 | 2 | 0 | 0 | 11 | [phase-3.8](s2/stages/phase-3.8.md) |
| 3.9 | Platform integration | 1 | 0 | 0 | 0 | 0 | 0 | [phase-3.9](s2/stages/phase-3.9.md) |
| **Total** | | **25** | **157** | **144** | **18** | **8** | **327** | |

> **Note:** `common-sys` (the 26th crate) is consumed transitively by stages 3.3,
> 3.4, and 3.7 but has no standalone parity contracts — its safe API surface is
> verified through its consumer crates. Authoritative source:
> [s2/stages/README.md](s2/stages/README.md).

**Exit gate:** All parity TOMLs for every stage are green.

## S4/SERVING — Verification, Conformance, Deploy-Ready

Prove behavioral equivalence between Go oracle and Rust implementation.

**Location:** `s4/`

| Phase | Activity | Output |
| --- | --- | --- |
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
