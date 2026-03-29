# Phase 3.9 — Platform Integration

| | |
| --- | --- |
| Stage | 3.9 |
| Scope | Systemd service lifecycle, FIPS build variant |
| Entry condition | Stage 3.8 exit gate passed (9/9 Must green) |
| Exit condition | Platform-specific integration tests pass |

---

## Crates

| Order | Crate | Group | Must | Should | Total |
| --- | --- | --- | --- | --- | --- |
| 1 | `host-service` | host/ | 0 | 0 | 0 |

### Notes

Contracts authored during S3.9 when systemd integration tests are
written. No pre-existing parity contracts — this stage is defined
by platform-specific verification that cannot be templated from Go
test catalogs.

---

## Parity Gate

No pre-defined Must or Should contracts. Contracts emerge during
implementation based on:

- systemd `sd-notify` lifecycle (`READY=1`, `STOPPING=1`)
- Service install/uninstall flows
- FIPS build variant compilation and runtime behavior

Gate criteria: all contracts written during this stage must be green
before S3 exit gate and S4 entry.

---

## Key Deliverables

### `host-service`

- Systemd service install (probes `/run/systemd/system`, explicit
  fail if absent — sysv deferred to FC)
- Systemd unit template generation
- `sd-notify` integration (`READY=1` after first successful
  connection, `STOPPING=1` on shutdown)
- Service uninstall

### FIPS build variant

- Feature-flagged BoringSSL FIPS module per
  [ADR-002](../../adr/002-fips-ci-build-matrix.md)
- CI build matrix: standard + FIPS profiles
- Runtime verification that FIPS module is active when feature is
  enabled

---

## Invariants Active

| ID | Invariant | Enforcement |
| --- | --- | --- |
| LIFE-1 | `sd-notify READY=1` after first connection | Integration test |
| LIFE-2 | `sd-notify STOPPING=1` on graceful shutdown | Integration test |

---

## Risks

| Risk | Description | Mitigation |
| --- | --- | --- |
| R1.4 | Build-tag and feature-gated branches drift from Linux-only | Linux-only CI profile; compile-time check table |

---

## Session Guidance

- **Session count:** 1 session
- **Cognitive load:** Low — straightforward systemd integration;
  FIPS is primarily a build-system concern
- **Platform requirement:** Must run on a Linux host with systemd
  ≥ 247 to exercise `sd-notify`
- **CI:** This stage produces the FIPS build matrix configuration
  for S2.9 environment
- **Cargo check:** `cargo check --workspace` +
  `cargo check --workspace --features fips`
