# ADR-010 - CLI Compatibility Mode

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Deferred — FC |
| Origin | Layer 6 in [dependency-decisions](../dependency-decisions.md), `cli-compat = []` workspace feature flag |
| Risk linkage | — |

## Question

Should [cfdrs](../../../../README.md) implement a cli-compat mode that matches urfave/cli
behavioral cadence for flag parsing edge cases?

## Decision

Deferred. `cli-compat` feature flag is defined in the workspace.
The default build uses `cli-native` (idiomatic [clap](https://crates.io/crates/clap) derive). cli-compat is
an FC-phase implementation option only, informed by S4 parity findings.
S4 is verification-only and does not open an implementation window.

## Rationale and Evidence

urfave/cli and [clap](https://crates.io/crates/clap) have different edge-case behaviors around flag
ordering, shorthand aliasing, and help text formatting. These differences are
unlikely to surface in primary tunnel operation but could appear in
operator-facing parity tests. Implementing cli-compat speculatively wastes S3
capacity, while implementing it during S4 would violate phase boundaries
(S4 is for verification). Evidence:

- [dependency-decisions](../dependency-decisions.md) Layer 6
- [scope](../scope.md) Could section
- [cmd/cloudflared/flags/flags](../../s1/atoms/cmd/cloudflared/flags/flags.md)

## Consequences

- S4 parity harness must include CLI invocation fixtures covering
  flag ordering and shorthand behavior and publish gap reports.
- No cli-compat implementation is allowed in S4.
- If a material CLI parity gap remains, implementation is scheduled in FC.
- If no material gap is found, cli-compat is closed as unnecessary.
