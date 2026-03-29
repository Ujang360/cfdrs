# ADR-002 - FIPS CI Build Matrix

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Deferred — S4/S5 |
| Origin | Layer 3.3 in [dependency-decisions](../dependency-decisions.md) |
| Risk linkage | — |

## Question

What CI build matrix is required to validate FIPS-mode builds,
and which pipeline stages gate on FIPS parity?

## Decision

Deferred to S4/S5. The `fips = []` feature flag is defined in the
workspace. The non-FIPS baseline is the only active build target in S3.

## Rationale and Evidence

FIPS is a compile-time-only switch — the Go baseline uses a `fips`
build tag that swaps the entire TLS backend. This is Rank 6 build-tag friction
per the porting-friction catalog. Attempting to CI-gate FIPS before the
non-FIPS baseline is proven adds risk without parity benefit. Evidence:

- [dependency-decisions](../dependency-decisions.md) § 3.3
- [platform-substrates](../../s1/catalogs/cross-cutting/platform-substrates.md)
  FIPS Build-Feature Matrix
- [fips/fips](../../s1/atoms/fips/fips.md)
- [fips/nofips](../../s1/atoms/fips/nofips.md)
- [scope](../scope.md) Won't section

## Consequences

- S4 planning must include a FIPS CI environment decision. Until
  then, `cargo build --features fips` must not be part of any required CI gate.
