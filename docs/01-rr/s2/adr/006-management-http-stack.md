# ADR-006 - Management HTTP Stack

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Decided |
| Origin | Layer 3.6 in [dependency-decisions](../dependency-decisions.md) |
| Risk linkage | R6.4 in [risks](../risks.md) |

## Question

Should [cfdrs](https://github.com/Ujang360/cfdrs) management endpoints use a full web framework abstraction by default, or use a minimal raw HTTP stack tailored to the management surface only?

## Decision

Use a minimal HTTP stack for management endpoints in RR scope. Permit middleware only for management concerns such as timeout and CORS. Do not introduce framework abstractions into proxy or session hot paths.

## Alternatives Considered

- Full framework-oriented management stack.
- Minimal HTTP server with constrained middleware on management routes only.

## Rationale and Evidence

- [dependency-decisions](../dependency-decisions.md) explicitly scopes management surface to `/ping`, `/host_details`, `/metrics`, and `/logs` (WebSocket), and states middleware must never be on proxy or session hot paths.
- [risks](../risks.md) records R6.4: unresolved management stack choice delays management and auth stage work.
- [management/service atom](../../s1/atoms/management/service.md) is critical path and sensitive to lifecycle and goroutine policy, not to broad framework features.
- A minimal stack reduces complexity and keeps boundaries explicit while preserving required management behavior.

## Consequences

- S2.6 should define a management-server module with explicit route handlers and bounded middleware.
- S3.7 implementation should avoid framework-specific global state patterns.
- WebSocket log streaming remains a management-only concern and must reuse shared logging event contracts.

## Scope Notes

- This ADR does not remove future framework adoption outside RR.
- Any expanded framework usage after RR requires a new ADR.
