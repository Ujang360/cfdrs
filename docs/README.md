# cfdrs — Documentation

Rust rewrite of [cloudflare/cloudflared](https://github.com/cloudflare/cloudflared),
baseline-pinned to tag `2026.3.0`.

## Project Lifecycle

```text
RR  Rust Rewrite          ← active, S1/RAW closed
FC  Feature Complete      ← TBD
UM  Upstream Maintenance  ← TBD
```

Each lifecycle phase is independent, long-lived, and produces its own
documentation stratum set. The repo is designed to outlive all three.

## Phases

| # | Code | Name | Status |
|---|------|------|--------|
| 1 | [RR](01-rr/README.md) | Rust Rewrite | Active — S2/SUBSTRATE next |
| 2 | [FC](02-fc/README.md) | Feature Complete | TBD |
| 3 | [UM](03-um/README.md) | Upstream Maintenance | TBD |

## Strata Map

```text
RR ─────────────────────────────────────────────────────────────────
│
├── S1/RAW          ✅ CLOSED
│   241 atoms · 30 catalogs · 100% coverage · Gini 0.3464
│   47 hub atoms · 8 Jaccard clusters · tests oracle
│   └── exit gate: PASSED
│
├── S2/SUBSTRATE    ⏳ NEXT
│   Dependencies (2 layers) · Scope · Risks · ADRs · Invariants
│   Architecture · Parity harness · Stages · Environment · Coherency
│   └── exit gate: full traceability chain
│
├── S3/COOK         ○ NOT STARTED
│   10 stages · critical path first · red→green parity progression
│   One crate · one session · one concern
│   └── exit gate: all parity tests green
│
└── S4/SERVING      ○ NOT STARTED
    Parity · Diff · Edge cases · Fuzz · Perf · Platform · Sign-off
    └── exit gate: release artifact

FC ─────────────────────────────────────────────────────────────────
    TBD — unscoped features from RR

UM ─────────────────────────────────────────────────────────────────
    TBD — upstream delta from 2026.3.0 forward
```
