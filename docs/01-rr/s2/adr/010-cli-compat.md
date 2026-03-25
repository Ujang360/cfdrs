# ADR-010 - CLI Compatibility Mode

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Deferred — S4 only if parity gap found |
| Origin | Layer 6 in [dependency-decisions](../dependency-decisions.md), `cli-compat = []` workspace feature flag |
| Risk linkage | — |

## Question

Should [cfdrs](https://github.com/Ujang360/cfdrs) implement a cli-compat mode that matches urfave/cli
behavioral cadence for flag parsing edge cases?

## Decision

Deferred. `cli-compat` feature flag is defined in the workspace.
The default build uses `cli-native` (idiomatic [clap](https://crates.io/crates/clap) derive). cli-compat is
activated only if S4 parity tests reveal a behavioral divergence in flag parsing
that cannot be resolved within the [clap](https://crates.io/crates/clap) model.

## Rationale and Evidence

urfave/cli and [clap](https://crates.io/crates/clap) have different edge-case behaviors around flag
ordering, shorthand aliasing, and help text formatting. These differences are
unlikely to surface in primary tunnel operation but could appear in
operator-facing parity tests. Implementing cli-compat speculatively wastes S3
capacity. Evidence:

- [dependency-decisions](../dependency-decisions.md) Layer 6
- [scope](../scope.md) Could section
- [cmd/cloudflared/flags/flags](../../s1/atoms/cmd/cloudflared/flags/flags.md)

## Consequences

- S4 parity harness must include CLI invocation fixtures covering
  flag ordering and shorthand behavior. If a gap is found, cli-compat is
  implemented as a feature-flagged layer. If no gap is found, cli-compat is
  permanently closed as unnecessary.
