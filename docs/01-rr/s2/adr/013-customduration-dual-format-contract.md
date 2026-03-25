# ADR-013 - CustomDuration Dual-Format Contract

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Decided |
| Origin | Layer 6 in [dependency-decisions](../dependency-decisions.md), R1.6 in [risks](../risks.md) |
| Risk linkage | R1.6 (P1) |

## Question

How should [cfdrs](../../../../README.md) preserve parity for duration serialization where JSON expects integer seconds but YAML expects Go duration strings?

## Decision

Implement explicit dual-format serialization behavior for the configuration duration type:

- JSON format: integer seconds.
- YAML format: Go-style duration string.

Round-trip tests for both formats are mandatory before S3.1 configuration completion.

## Alternatives Considered

- Normalize both formats to a single representation.
- Preserve Go-compatible dual-format behavior with explicit serializers.

## Rationale and Evidence

- [risks](../risks.md) identifies R1.6 as a parity-risking mismatch with silent behavior drift.
- [dependency-decisions](../dependency-decisions.md) documents human-readable duration handling and key timeout surfaces.
- [config/model atom](../../s1/atoms/config/model.md) anchors config contract behavior.
- Preserving format-specific semantics prevents hidden timeout mutation during config translation workflows.

## Consequences

- S3.1 config crate must provide format-specific serializers and deserializers.
- Phase 2.7 parity design should include JSON and YAML round-trip fixtures for representative timeout values.
- Documentation must state format behavior explicitly to avoid tooling assumptions.

## Scope Notes

- This ADR covers duration encoding and decoding only.
- It does not change timeout defaults or introduce new timeout fields.
