# ADR-017 - Access S4 Drop-In Replacement Contract

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Decided |
| Origin | Access reclassification rationale in [scope](../scope.md) |
| Risk linkage | R6.6 (scope to parity leakage) |

## Question

What exact parity expectation is meant by the S2.2 statement that Access commands are required by the S4 drop-in replacement contract?

## Decision

Define the S4 drop-in replacement contract for Access as follows:

- Access command surfaces classified as Should in [scope](../scope.md) must be present and behaviorally equivalent at S4 sign-off.
- Behavioral equivalence is validated against parity harness scenarios, not only CLI shape.
- Any intentionally deferred Access behavior must be listed explicitly in S4 acceptance criteria.

## Alternatives Considered

- Treat Access as optional despite Should classification.
- Formalize Access parity contract as an explicit S4 requirement.

## Rationale and Evidence

- [scope](../scope.md) reclassified multiple Access atoms from Could to Should with explicit drop-in replacement rationale.
- [risks](../risks.md) R6.6 warns that scope boundaries can leak into parity expectations if not codified.
- [docs/01-rr/README](../../README.md) defines S4 sign-off as behavioral equivalence and full scoped-feature coverage.
- Making this contract explicit prevents misalignment across S2.7 parity design and S4 sign-off.

## Consequences

- S2.7 parity harness design must include Access scenarios tied to Should-scoped atoms.
- S4 sign-off checklist must include an explicit Access parity section.
- Any Access exclusion requires documented acceptance, not implicit omission.

## Scope Notes

- This ADR sets verification and acceptance boundaries, not implementation detail for Access internals.
- It does not upgrade all Access features to Must tier.
