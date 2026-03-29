# Phase 3.2 — Control Plane

| | |
| --- | --- |
| Stage | 3.2 |
| Scope | Cap'n Proto RPC wire encoding, registration client/server |
| Entry condition | Stage 3.0 exit gate passed (6/6 Must green) |
| Exit condition | All 10 Must-tier contracts green |

---

## Crates

| Order | Crate | Group | Must | Should | Total |
| --- | --- | --- | --- | --- | --- |
| 1 | `tunnel-rpc` | tunnel/ | 10 | 0 | 10 |

### Notes

Single-crate stage. `tunnel-rpc` is pure encoding/decoding — no
lifecycle, no state machine. Depends only on `tunnel-core` (from
3.0) for shared types.

---

## Parity Gate

**Must-tier contracts (10):**

- `REG.capnp_connection_options_roundtrip` — Cap'n Proto wire format
- `REG.registration_rpc_success` — registration success path
- `REG.registration_rpc_error` — error path
- `REG.registration_rpc_retryable` — retryable error classification
- `REG.connect_request_roundtrip` — connect request serialization
- `REG.udp_session_*` — UDP session registration datagrams
- `REG.manage_configuration_rpc` — configuration management RPC

All 10 must be `green` before stage 3.3 `tunnel-connection` begins
(which imports `tunnel-rpc`).

---

## Key Deliverables

- Cap'n Proto schema compilation (`tunnelrpc.capnp`,
  `quic_metadata_protocol.capnp`)
- POGS types: `RegistrationOptions`, `TunnelRegistration`,
  `UpdateConfigurationRequest`, `UpdateConfigurationResponse`,
  `UDPSessionRegistrationDatagram` (v2/v3)
- Session manager RPC client and server stubs
- RPC metrics

---

## Invariants Active

| ID | Invariant | Enforcement |
| --- | --- | --- |
| REG-1 | Registration RPC success includes tunnel UUID | Contract test |

---

## Risks

| Risk | Description | Mitigation |
| --- | --- | --- |
| R3.5 | RPC retryable/non-retryable edge implemented inconsistently | Centralize mapping in `tunnel-rpc` |
| R5.1 | Cap'n Proto wire format must match Go exactly | Byte-level round-trip contracts |

---

## Session Guidance

- **Session count:** 1 session
- **Cognitive load:** Moderate — Cap'n Proto schema translation
  requires careful attention to field ordering and defaults
- **Parallelism:** Can run in parallel with stage 3.1
- **Cargo check:** `cargo check --workspace` after completion
