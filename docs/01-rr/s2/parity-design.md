# S2.7 — Parity Harness Design

|          |                                                                                                                                                   |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| Phase    | 2.7                                                                                                                                               |
| Status   | Active                                                                                                                                            |
| Consumes | [scope](scope.md), [risks](risks.md), [ADR index](adr/README.md), [invariants](invariants.md), [architecture](architecture.md), S1 tests catalogs |
| Produces | S3 red-by-default parity contracts, S4 execution model                                                                                            |

Design of the parity testing framework for cfdrs: a TOML contract
schema, oracle binary integration model, and per-stage coverage matrix
that maps all Go behavioral contracts + invariants + fuzz targets into
red-by-default parity tests.

## Table of Contents

- [TOML Contract Schema](#toml-contract-schema)
- [Oracle Interaction Model](#oracle-interaction-model)
- [Red-by-Default Methodology](#red-by-default-methodology)
- [ADR Parity Obligations](#adr-parity-obligations)
- [Scope Exclusions and Parity Boundaries](#scope-exclusions-and-parity-boundaries)
- [Contract Catalog by S3 Stage](#contract-catalog-by-s3-stage)
- [Coverage Matrix](#coverage-matrix)
- [Fuzz Parity Strategy](#fuzz-parity-strategy)
- [S4 Execution Model](#s4-execution-model)
- [Exit Gate](#exit-gate)

---

## TOML Contract Schema

### Design Rationale

The parity contract schema serves two consumers with different needs:

| Consumer           | Need                                       | Implication                                                |
| ------------------ | ------------------------------------------ | ---------------------------------------------------------- |
| **S3 implementor** | Know what to implement and test, per-crate | Contracts grouped by crate, linked to atoms and invariants |
| **S4 test runner** | Machine-executable parity specs            | Oracle command, expected behavior, comparison mode         |

The schema captures the *parity boundary* — the explicit line between
"this behavior must match Go exactly" and "this behavior is allowed to
differ."

### File Organization

```text
docs/01-rr/s2/parity/
  parity-design.md              — this document
  contracts/
    {crate-name}.toml           — one file per S2.6 crate (26 files)
  exclusions/
    wont-tier.toml              — Won't atoms (8) with skip reasons
    could-tier.toml             — Could atoms (18) with skip reasons
    rr-deferred.toml            — RR-deferred features (12)
    permanent.toml              — Permanently excluded (6)
```

Example: `docs/01-rr/s2/parity/contracts/tunnel-session.toml`

**Why per-crate, not per-stage?** S3 rule is "one crate, one session."
The implementor needs all contracts for their crate in one file. Stage
assignment is a field on each contract, not a file boundary.

**Why under docs/?** These are S2 design artifacts (specifications),
not runtime test files. S3 reads from these. Actual Rust test code
lives in each crate's `tests/`. S4 oracle execution lives in
`app/tests/e2e/`.

### Schema Definition

```toml
# File-level metadata
[meta]
crate = "tunnel-session"           # S2.6 crate name (from architecture.md)
s3_stages = ["3.4"]                # S3 stages that consume this file
atom_count = 4                     # number of S1 atoms this crate covers
contract_count = 12                # total contracts in this file

# Repeatable contract block — one per behavioral claim
[[contract]]
id = "SESSION.idle_timeout_reset"  # {INVARIANT_PREFIX}.{snake_case_name}
description = """
Session idle timeout resets whenever data flows through the session.
A session with continuous activity must not be closed by the idle
timer, regardless of absolute elapsed time.
"""

# --- Classification ---
category = "scenario"              # unit | integration | e2e | fuzz
parity_level = "behavioral"        # exact | behavioral | semantic
tier = "must"                      # must | should | could
status = "red"                     # red | green | skip
skip_reason = ""                   # required when status = "skip"

# --- Traceability ---
[contract.trace]
s1_atoms = ["quic/v3/session"]
s1_tests = ["TestSessionIdleTimeout", "TestSessionActivity"]
s2_invariant = "SESSION-1"         # empty string if no direct invariant
s2_risks = ["R2.2"]                # risk IDs this contract mitigates
s2_adrs = ["ADR-016"]              # ADR IDs this contract satisfies

# --- Oracle specification ---
[contract.oracle]
type = "behavioral"                # oracle-comparable | oracle-captured |
                                   # behavioral | property
deterministic = true               # false if Go output has non-deterministic
                                   # elements
timeout_ms = 5000                  # max execution time for oracle subprocess

# Oracle type determines which fields below are used:
#
# oracle-comparable: run Go binary, run Rust binary, compare outputs
#   → uses: command, args, env, comparison
#
# oracle-captured: compare against pre-recorded Go output artifacts
#   → uses: capture_file, comparison
#
# behavioral: no direct oracle comparison; Rust test encodes the contract
#   → uses: (none — test code is the contract)
#
# property: stateless property test, no oracle needed
#   → uses: (none — proptest stub is the contract)

command = ""                       # oracle binary subcommand
args = []                          # CLI arguments
env = {}                           # environment variables

# Pre-recorded oracle artifact path (if type = oracle-captured)
capture_file = ""                  # relative to oracle/captures/

# --- Comparison specification ---
[contract.comparison]
mode = "exit_code"                 # exit_code | stdout | stderr | json_path |
                                   # proto_diff | file_diff | regex | custom
# Mode-specific fields:
exit_code = 0                      # expected exit code (mode = exit_code)
stdout_pattern = ""                # regex pattern (mode = regex)
json_paths = []                    # JSONPath expressions (mode = json_path)
proto_schema = ""                  # .capnp schema path (mode = proto_diff)
tolerance = ""                     # for approximate comparisons (e.g., "±5ms")
```

### Parity Levels

Three levels handle the spectrum from deterministic to
non-deterministic Go behavior:

| Level          | Definition                                                 | When to use                                              | Comparison approach                                 |
| -------------- | ---------------------------------------------------------- | -------------------------------------------------------- | --------------------------------------------------- |
| **exact**      | Byte-for-byte identical output                             | Serialization round-trips, wire frames, exit codes       | Literal diff                                        |
| **behavioral** | Same observable behavior, different representation allowed | State machines, lifecycle ordering, error classification | Semantic comparison (exit code + structured output) |
| **semantic**   | Same intent, implementation-specific differences accepted  | Concurrency timing, log ordering, metric naming          | Pattern matching + invariant checks                 |

### Oracle Types

Four types reflect how each contract relates to the Go oracle binary:

| Type                  | Oracle involvement                                            | Count estimate | Example                                                             |
| --------------------- | ------------------------------------------------------------- | -------------- | ------------------------------------------------------------------- |
| **oracle-comparable** | Run Go binary → run Rust binary → diff                        | ~40            | CLI flag parsing, config serialization, exit codes                  |
| **oracle-captured**   | Pre-record Go output → compare at test time                   | ~60            | Cap'n Proto frames, wire captures, HTTP responses                   |
| **behavioral**        | No oracle subprocess; Rust test encodes the contract directly | ~140           | State machine transitions, lifecycle ordering, concurrency behavior |
| **property**          | No oracle; pure property test (proptest)                      | ~40            | Idempotency, determinism, exhaustiveness, round-trip                |

Most Go behavioral contracts (~280) cannot be tested by running the Go
binary as a subprocess — they test internal behavior (struct methods,
interface contracts, state machines). Only CLI-surface and serialization
contracts are oracle-comparable. The majority become `behavioral`
contracts where the Rust test code *is* the parity spec.

### Contract ID Namespace

Contract IDs reuse the S2.5 invariant prefix namespace plus a
short snake_case name:

```text
{PREFIX}.{descriptive_name}

TRANS.quic_connection_serve_until_unregistered
SESSION.idle_timeout_reset
CONFIG.customduration_json_roundtrip
PROXY.rule_match_deterministic
FUZZ.datagram_v3_decode_no_panic
```

Contracts that directly test an S2.5 invariant use the invariant ID
as the contract ID prefix (e.g., `SESSION-1` → `SESSION.idle_timeout_reset`).

Contracts from S1 Go tests that do not map to an S2.5 invariant use the
same prefix taxonomy but have `s2_invariant = ""`.

Fuzz contracts use the `FUZZ` prefix regardless of domain.

### Contract Derivation Rules

When a single Go test function covers multiple distinct behaviors,
the contract author splits it into N separate TOML contracts.

**Splitting rules:**

1. If a Go test exercises N distinct behavioral claims → N contracts
2. If a Go test is a helper, setup, or benchmark → no contract
3. Table-driven Go tests with N sub-cases may become 1 contract if
   the sub-cases test the same behavioral property, or N contracts if
   they test distinct properties

Each contract's `[contract.trace]` block includes `s1_tests` listing
the Go test function(s) it derives from. When splitting occurs, all
resulting contracts reference the same Go test. A `derivation_note`
comment in the TOML documents the split rationale:

```toml
[[contract]]
id = "SESSION.idle_timeout_reset"
# derivation: split from TestSessionManager (behavior 1 of 3)
# TestSessionManager also covers: SESSION.register_new_session,
#   SESSION.unregister_closes_session

[contract.trace]
s1_tests = ["TestSessionManager"]
```

---

## Oracle Interaction Model

### Binary Placement

```text
oracle/
  cloudflared-amd64-linux     # Go binary at tag 2026.3.0 (gitignored)
  README.md                   # hash, source, acquisition instructions
  captures/                   # pre-recorded oracle outputs (git-tracked)
    rpc/                      # Cap'n Proto frame captures
    config/                   # serialization round-trip fixtures
    cli/                      # CLI output captures
    fuzz/                     # fuzz corpus seeds (git-tracked)
```

- `oracle/cloudflared-amd64-linux` is **gitignored** (binary, ~50MB)
- `oracle/captures/` is **git-tracked** (text/binary fixtures, small)
- [oracle/README.md](../../../oracle/README.md) documents: source tag,
  SHA-256 hash, build command, platform requirements, acquisition steps

### Invocation Model

Parity tests that use the oracle binary follow a subprocess model:

```text
┌─────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Test code   │────→│  Oracle runner    │────→│  Go binary       │
│  (Rust)      │     │  (subprocess)     │     │  (subprocess)    │
│              │←────│                   │←────│                  │
│  Comparison  │     │  Captured output  │     │  stdout/stderr   │
└─────────────┘     └──────────────────┘     └──────────────────┘
```

**Oracle runner responsibilities:**

1. **Environment isolation:** Clean env, temp directories, no leaked
   state
2. **Process management:** Fork, set timeout, capture stdout/stderr/exit
   code
3. **Network isolation:** Loopback-only, fixed ports where needed
4. **Determinism enforcement:** Fixed seed env vars, time injection
   where the Go binary supports it
5. **Output normalization:** Strip timestamps, PIDs, and other
   non-deterministic fields before comparison

**Determinism classification:**

| Contract property    | Deterministic? | Normalization needed                 |
| -------------------- | -------------- | ------------------------------------ |
| Exit code            | Yes            | None                                 |
| CLI help text        | Yes            | None                                 |
| Config serialization | Yes            | None                                 |
| Cap'n Proto frames   | Yes            | None (binary-stable)                 |
| Log output           | No             | Strip timestamp, PID, goroutine ID   |
| Metric values        | No             | Strip timestamps, normalize counters |
| Connection timing    | No             | Use tolerance windows                |
| Error messages       | Partially      | Normalize Go-specific strings        |

### Capture Mode for oracle-captured Contracts

For contracts where running the Go binary at test time is impractical
(needs network, edge connection, etc.), pre-recorded captures are used:

1. **Recording phase** (one-time, human-supervised):
   - Run Go binary in controlled environment
   - Capture output to `oracle/captures/{domain}/{contract_id}.{ext}`
   - Extensions: `.bin` (binary), `.json`, `.txt`, `.capnp` (Cap'n
     Proto)
   - Document capture conditions in companion `.md` file

2. **Test phase** (automated, every CI run):
   - Rust test loads capture file
   - Runs Rust code with same inputs
   - Compares Rust output against capture using contract's comparison
     mode

3. **Refresh policy:**
   - Captures are pinned to oracle binary hash
   - If oracle binary is ever rebuilt, all captures must be re-recorded
   - Capture freshness is verified by hash check in CI

### Wire Capture Format (for R5.2 Cap'n Proto Validation)

Risk R5.2 requires validating Cap'n Proto request/response frames:

```text
oracle/captures/rpc/
  registration_request.capnp.bin    # raw Cap'n Proto bytes
  registration_response.capnp.bin
  config_update.capnp.bin
  reconnect.capnp.bin
  README.md                         # capture conditions, schema version
```

Capture method options:

- Instrument Go binary's RPC layer to dump raw Cap'n Proto bytes
- Use a MITM proxy between cloudflared and edge (loopback)
- Store raw bytes + schema reference for Rust-side decode validation

**Feature flag:** `cfg(feature = "parity-capture")` in the Rust binary,
`--capture-rpc-frames` CLI flag on the Go oracle. Captures are
**live-generated** in CI, not pre-recorded static fixtures. The capture
feature flag is stripped from release builds. Cap'n Proto frames are
binary-stable → exact comparison is valid. Requires CI environment with
tunnel token → runs in separate CI job, not blocking per-PR merges.

---

## Red-by-Default Methodology

### Lifecycle States

```text
  ┌─────┐    S3 implements    ┌───────┐    S4 verifies    ┌──────────┐
  │ RED  │───────────────────→│ GREEN  │───────────────────→│ VERIFIED │
  └─────┘                    └───────┘                    └──────────┘
    │                            │
    │  out of scope              │  regression
    ▼                            ▼
  ┌──────┐                   ┌─────┐
  │ SKIP │                   │ RED  │ ← CI blocks merge
  └──────┘                   └─────┘
```

| Status     | Meaning                                              | Who sets it                   | Gate                      |
| ---------- | ---------------------------------------------------- | ----------------------------- | ------------------------- |
| `red`      | Contract exists, test not yet implemented or failing | S2.7 author (initial)         | S3 must turn green        |
| `green`    | Rust test passes, parity claim satisfied             | S3 implementor                | CI enforces no regression |
| `skip`     | Explicitly out of scope with justification           | S2.7 author or S3 implementor | Requires `skip_reason`    |
| `verified` | S4 oracle comparison confirms parity                 | S4 verifier                   | S4 sign-off gate          |

### CI Enforcement Rules

1. **No green-to-red regression:** If a contract was `green` in the
   previous commit and is now `red`, CI fails the build.
2. **Skip requires reason:** Every `status = "skip"` must have a
   non-empty `skip_reason` field.
3. **Stage gate (Must-tier):** Before moving to stage N+1, all
   **Must-tier** contracts assigned to stage N must be `green` or
   `skip`. Must gates unblock the next stage.
4. **Stage gate (Should-tier):** Should-tier contracts do NOT block
   stage advancement. They start `red`, are visible in the report
   with a separate progress bar, and can turn green in any order
   (jagged implementation). Should contracts remaining `red` at S3
   exit gate require a documented acceptance note.
5. **Coverage check:** CI verifies that the total contract count
   matches the expected count from the coverage matrix.

### Red-by-Default Workflow

**S2.7 (now):** Author writes all TOML contracts with
`status = "red"`. Every contract is a specification, not a test.

**S3 (future, per-crate):**

1. Implementor opens `parity/{crate}.toml`
2. Reads contract descriptions and oracle specs
3. Writes Rust test code that satisfies each contract
4. Flips `status = "red"` → `status = "green"` in the TOML
5. CI verifies: no regressions, stage gate satisfied

**S4 (future):**

1. Run oracle comparison for all `oracle-comparable` contracts
2. Verify captures for all `oracle-captured` contracts
3. Flip `status = "green"` → `status = "verified"` when oracle confirms
4. Generate aggregate report: green/red/skip/verified counts per crate

### Aggregate Report Format

Report shows BOTH views — per-stage with tier breakdown and tier rollup.

**Primary view — per-stage with tier breakdown:**

```text
S3 Parity Report — 2026-XX-XX
═══════════════════════════════════════════════════════
Stage 3.0: 24/24 green (100%)  ████████████████████ ✓
  Must:   18/18 ✓  Should: 6/6 ✓
Stage 3.1: 36/44 green (82%)   ███████████████░░░░ …
  Must:   28/28 ✓  Should: 8/16 …
Stage 3.2:  0/18 green (0%)    ░░░░░░░░░░░░░░░░░░░ ✗
  Must:    0/12 ✗  Should: 0/6 ✗
...
```

**Secondary view — tier rollup:**

```text
Must-tier:    46/157 green (29%)  ████████░░░░░░░░░░░░
Should-tier:  14/144 green (10%)  ██░░░░░░░░░░░░░░░░░░
Invariants:   12/29 covered       █████████░░░░░░░░░░░
Fuzz:          0/8 implemented    ░░░░░░░░░░░░░░░░░░░░
```

**Unblocking rule:** A stage is unblocked for the NEXT stage when
all its **Must** contracts are green. Should contracts are jagged —
they can turn green in any order, even after downstream stages
start.

### Contract Authoring Rules

1. **One contract per behavioral claim.** If a Go test function tests
   three distinct behaviors, write three contracts.
2. **Description is the contract.** The natural-language description
   must be precise enough that an implementor can write the test
   without reading S1. S2.5 invariant stubs provide additional detail.
3. **Oracle type determines test shape.** Do not force oracle comparison
   on contracts where it is impractical — use `behavioral` or `property`
   type instead.
4. **Parity level is conservative.** Default to `exact` for
   serialization, `behavioral` for state machines, `semantic` for
   concurrency. Never claim `exact` if Go output has non-deterministic
   elements.
5. **Traceability is mandatory.** Every contract must link to at least
   one S1 atom. Links to S2 invariants, risks, and ADRs are optional
   but strongly encouraged.

---

## ADR Parity Obligations

Six ADRs contain explicit S2.7 deferral clauses requiring parity
fixture design:

| ADR                                                             | S2.7 Obligation                                                                                       | Required Fixtures                                                                       | Crate(s)                          | Category                |
| --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | --------------------------------- | ----------------------- |
| [ADR-005](adr/005-retry-backoff-library-vs-custom.md)           | Explicit retry-sequence and reset-case fixtures                                                       | Retry delay bounded + reproducible, reset-restores-base                                 | common-retry                      | property                |
| [ADR-013](adr/013-customduration-dual-format-contract.md)       | JSON and YAML round-trip fixtures for representative timeout values                                   | CustomDuration round-trip JSON (integer seconds), round-trip YAML (Go duration strings) | common-wire-primitives            | exact                   |
| [ADR-014](adr/014-startup-dag-contract.md)                      | Startup-order-sensitive scenarios                                                                     | Phase ordering verification, sd-notify fires iff all phases succeed                     | app                               | scenario                |
| [ADR-017](adr/017-access-s4-drop-in-contract.md)                | Access scenarios tied to Should-scoped atoms; S4 sign-off must include explicit Access parity section | Access command behavioral equivalence on each Should-tier Access atom                   | operator-cli-native, common-token | behavioral              |
| [ADR-018](adr/018-quic-connection-ownership-enforcement.md)     | Affinity misuse safeguards                                                                            | Connection assignment to single thread, `!Send` enforcement                             | tunnel-transport                  | compile-time + property |
| [ADR-019](adr/019-cancellation-timeout-propagation-contract.md) | Timeout and cancellation fixtures across registration, control stream, and session transitions        | Cancellation propagation across boundary, timeout propagation across boundary           | tunnel-connection, tunnel-session | scenario                |

---

## Scope Exclusions and Parity Boundaries

### Won't-Tier Atoms (8)

Permanently excluded from parity. Encoded as informational `skip`
entries in the crate TOML that would otherwise own them:

| Atom                                | Would-be Crate      | Skip Reason                                            |
| ----------------------------------- | ------------------- | ------------------------------------------------------ |
| cmd/cloudflared/macos_service       | host-service        | won't-tier: outside RR platform boundary (macOS)       |
| cmd/cloudflared/updater/* (5 atoms) | operator-cli-native | won't-tier: auto-update excluded from RR scope         |
| cmd/cloudflared/windows_service     | host-service        | won't-tier: outside RR platform boundary (Windows)     |
| fips/fips                           | common-sys          | won't-tier: FIPS feature gating excluded from RR scope |

### Could-Tier Atoms (18)

Deferred. No contracts needed for S3. May be promoted to S4 acceptance
tests if scope changes. `skip_reason = "could-tier-deferred: {rationale}"`

### RR-Deferred Features (12)

Key exclusion rules:

- **HTTP/2 transport:** All connection/http2 contracts → `skip` with
  reason "http2-deferred-per-ADR-001"
- **Non-Linux platforms:** All platform-gated contracts → `skip` with
  reason "non-linux-deferred"
- **FIPS:** All FIPS-specific contracts → `skip` with reason
  "fips-deferred"
- **QUIC 0-RTT:** Any 0-RTT contracts → `skip` with reason
  "0rtt-deferred"

### Permanently Excluded (6)

Features that need no TOML presence — they are not Go behaviors to
track. Mentioned only in this exclusion register.

### Access Parity Boundary ([ADR-017](adr/017-access-s4-drop-in-contract.md))

**In scope for S3/S4 parity:**

- All Access atoms classified as Should in [scope](scope.md):
  - token/token (acquisition, file-lock retry)
  - token/transfer (org-to-app exchange, browser launch)
  - credentials/credentials (account/zone ID, API token, cfapi.Client)
  - credentials/origin_cert (PEM/JSON round-trip, cert discovery)
  - sshgen/sshgen (SSH key generation)
  - cmd/cloudflared/access/* subcommands in Should tier
- Parity level: **behavioral** (CLI output format may differ)
- Oracle type: **oracle-comparable** for CLI surface, **behavioral**
  for internal token logic

**Explicitly deferred (with documented acceptance):**

- cmd/cloudflared/tunnel/login — Could tier, deferred
- Access subcommand teamnet/vnets admin — Could tier, deferred
- Any Access behavior requiring live Cloudflare API —
  `oracle-captured` with pre-recorded fixtures

---

## Contract Catalog by S3 Stage

This section summarizes the contract catalog derived from S1 Go test
catalogs across Batches C and D. Detailed per-contract tables are in
the individual TOML files under `parity/contracts/`.

### Stage 3.0 — Foundation

| Crate                  | Must  | Should | Skip  | Fuzz  | Total |
| ---------------------- | ----- | ------ | ----- | ----- | ----- |
| common-wire-primitives | 6     | 0      | 0     | 0     | 6     |
| common-error           | 0     | 0      | 0     | 0     | 0     |
| tunnel-core            | 0     | 0      | 0     | 0     | 0     |
| **Subtotal**           | **6** | **0**  | **0** | **0** | **6** |

Key contracts: TRANS.header_serialize_roundtrip (HTTP header round-trip),
TRANS.websocket_accept_key_rfc6455 (RFC 6455 key generation).

### Stage 3.1 — Independent Crates

| Crate                | Must  | Should | Skip   | Fuzz  | Total  |
| -------------------- | ----- | ------ | ------ | ----- | ------ |
| common-signal        | 2     | 0      | 0      | 0     | 2      |
| common-retry         | 0     | 6      | 0      | 0     | 6      |
| common-observability | 0     | 13     | 0      | 1     | 14     |
| common-cfapi         | 0     | 7      | 11     | 0     | 18     |
| common-token         | 2     | 18     | 0      | 0     | 20     |
| config-core          | 2     | 0      | 0      | 0     | 2      |
| tunnel-metrics       | 0     | 2      | 0      | 0     | 2      |
| **Subtotal**         | **6** | **46** | **11** | **1** | **64** |

Key contracts: LIFE.signal_double_notify (safe double-close),
CONFIG.yaml_file_settings (config deserialization), CRED.credentials_read
(credential resolution).

### Stage 3.2 — Control Plane

| Crate        | Must   | Should | Skip  | Fuzz  | Total  |
| ------------ | ------ | ------ | ----- | ----- | ------ |
| tunnel-rpc   | 10     | 0      | 0     | 0     | 10     |
| **Subtotal** | **10** | **0**  | **0** | **0** | **10** |

Key contracts: REG.capnp_connection_options_roundtrip (Cap'n Proto wire
format), REG.registration_rpc_success/error/retryable (RPC lifecycle).

### Stage 3.3 — Transport

| Crate             | Must   | Should | Skip  | Fuzz  | Total  |
| ----------------- | ------ | ------ | ----- | ----- | ------ |
| tunnel-transport  | 13     | 3      | 6     | 0     | 22     |
| tunnel-connection | 17     | 5      | 0     | 0     | 22     |
| **Subtotal**      | **30** | **8**  | **6** | **0** | **44** |

Key contracts: TRANS.quic_server_\* (4 proxy types), TRANS.edge_\*
(8 edge discovery contracts), TRANS.protocol_selector_new (protocol
selection). 6 HTTP/2 contracts skipped per ADR-001.

### Stage 3.4 — Sessions

| Crate          | Must   | Should | Skip  | Fuzz  | Total  |
| -------------- | ------ | ------ | ----- | ----- | ------ |
| tunnel-session | 52     | 11     | 0     | 6     | 69     |
| **Subtotal**   | **52** | **11** | **0** | **6** | **69** |

Key contracts: SESSION.v3_\* (datagram wire format), SESSION.muxer_\*
(muxer lifecycle), SESSION.manager_\* (session manager),
SESSION.serve_migrate (session migration). 6 fuzz targets for datagram
decode robustness.

### Stage 3.5 — Configuration Runtime and Supervisor

| Crate             | Must  | Should | Skip  | Fuzz  | Total  |
| ----------------- | ----- | ------ | ----- | ----- | ------ |
| config-runtime    | 0     | 17     | 0     | 0     | 17     |
| tunnel-supervisor | 1     | 2      | 0     | 0     | 3      |
| **Subtotal**      | **1** | **19** | **0** | **0** | **20** |

Key contracts: LIFE.protocol_fallback_state_machine (QUIC→HTTP2
fallback + reset), CONFIG.update_configuration (orchestrator update).

### Stage 3.6 — Ingress Proxy

| Crate                | Must   | Should | Skip  | Fuzz  | Total  |
| -------------------- | ------ | ------ | ----- | ----- | ------ |
| tunnel-ingress-proxy | 37     | 26     | 0     | 1     | 64     |
| **Subtotal**         | **37** | **26** | **0** | **1** | **64** |

Key contracts: PROXY.single_origin_\* (HTTP/WS/SSE proxying),
PROXY.parse_ingress_table_driven (rule parsing), PROXY.rule_matches_\*
(hostname matching), PROXY.conn_\* (cross-protocol proxying), PROXY.socks5_\*
(SOCKS5 proxying).

### Stage 3.7 — Management and Diagnostics

| Crate             | Must  | Should | Skip  | Fuzz  | Total  |
| ----------------- | ----- | ------ | ----- | ----- | ------ |
| tunnel-management | 6     | 21     | 0     | 0     | 27     |
| host-diagnostic   | 0     | 11     | 1     | 0     | 12     |
| **Subtotal**      | **6** | **32** | **1** | **0** | **39** |

Key contracts: MGMT.read_events_loop (WebSocket streaming),
MGMT.start_stream_\* (session limits), MGMT.diag_\* (system info
collection).

### Stage 3.8 — Application and CLI

| Crate               | Must  | Should | Skip  | Fuzz  | Total  |
| ------------------- | ----- | ------ | ----- | ----- | ------ |
| app                 | 3     | 0      | 0     | 0     | 3      |
| operator-cli-native | 6     | 2      | 0     | 0     | 8      |
| operator-cli-common | 0     | 0      | 0     | 0     | 0      |
| operator-cli-compat | 0     | 0      | 0     | 0     | 0      |
| operator-cli        | 0     | 0      | 0     | 0     | 0      |
| **Subtotal**        | **9** | **2**  | **0** | **0** | **11** |

Key contracts: LIFE.manager_\* (overwatch service lifecycle),
PROXY.tunnel_\* (CLI subcommands, placeholder — refine during S3).

### Stage 3.9 — Platform Integration

| Crate        | Must  | Should | Skip  | Fuzz  | Total |
| ------------ | ----- | ------ | ----- | ----- | ----- |
| host-service | 0     | 0      | 0     | 0     | 0     |
| **Subtotal** | **0** | **0**  | **0** | **0** | **0** |

Contracts authored during S3.9 when systemd integration tests are
written.

---

## Coverage Matrix

### Per-Stage Summary

| S3 Stage  | Crates | Must    | Should  | Skip   | Fuzz  | Total   | Gate              |
| --------- | ------ | ------- | ------- | ------ | ----- | ------- | ----------------- |
| 3.0       | 3      | 6       | 0       | 0      | 0     | 6       | Must: 6/6         |
| 3.1       | 7      | 6       | 46      | 11     | 1     | 64      | Must: 6/6         |
| 3.2       | 1      | 10      | 0       | 0      | 0     | 10      | Must: 10/10       |
| 3.3       | 2      | 30      | 8       | 6      | 0     | 44      | Must: 30/30       |
| 3.4       | 1      | 52      | 11      | 0      | 6     | 69      | Must: 52/52       |
| 3.5       | 2      | 1       | 19      | 0      | 0     | 20      | Must: 1/1         |
| 3.6       | 1      | 37      | 26      | 0      | 1     | 64      | Must: 37/37       |
| 3.7       | 2      | 6       | 32      | 1      | 0     | 39      | Must: 6/6         |
| 3.8       | 5      | 9       | 2       | 0      | 0     | 11      | Must: 9/9         |
| 3.9       | 1      | 0       | 0       | 0      | 0     | 0       | n/a               |
| **Total** | **26** | **157** | **144** | **18** | **8** | **327** | **Must: 157/157** |

### Parity Level Distribution

| Parity Level | Count   | Percentage |
| ------------ | ------- | ---------- |
| exact        | ~78     | 24%        |
| behavioral   | ~225    | 69%        |
| semantic     | ~8      | 2%         |
| (skip)       | 18      | 5%         |
| **Total**    | **327** | **100%**   |

### Oracle Type Distribution

| Oracle Type       | Count         | Percentage |
| ----------------- | ------------- | ---------- |
| behavioral        | ~185          | 57%        |
| property          | ~78           | 24%        |
| oracle-captured   | ~1            | <1%        |
| oracle-comparable | ~0 (deferred) | 0%         |
| (skip)            | 18            | 5%         |
| (placeholder)     | ~37           | 11%        |
| **Total**         | **327**       | **100%**   |

Oracle-comparable contracts emerge in S4 when CLI output and config
serialization differences are compared against the Go oracle.

### Must-Tier Atom Coverage (33 atoms)

All 33 Must atoms from [scope](scope.md) have at least one
parity contract:

| #   | Must Atom                                | Crate                | Contracts                                                | Status |
| --- | ---------------------------------------- | -------------------- | -------------------------------------------------------- | ------ |
| 1   | cmd/cloudflared/flags/flags              | operator-cli-native  | PROXY.tunnel_tag_parsing                                 | ✓      |
| 2   | cmd/cloudflared/tunnel/cmd               | operator-cli-native  | PROXY.tunnel_cmd                                         | ✓      |
| 3   | cmd/cloudflared/tunnel/configuration     | operator-cli-native  | CONFIG.tunnel_cmd_config                                 | ✓      |
| 4   | cmd/cloudflared/tunnel/credential_finder | operator-cli-native  | PROXY.tunnel_subcommand_context                          | ✓      |
| 5   | config/configuration                     | config-core          | CONFIG.yaml_file_settings                                | ✓      |
| 6   | config/model                             | config-core          | CONFIG.origin_request_roundtrip                          | ✓      |
| 7   | credentials/credentials                  | common-token         | CRED.credentials_read, CRED.credentials_client           | ✓      |
| 8   | overwatch/app_manager                    | app                  | LIFE.manager_* (3)                                       | ✓      |
| 9   | overwatch/manager                        | app                  | LIFE.manager_* (3)                                       | ✓      |
| 10  | signal/safe_signal                       | common-signal        | LIFE.signal_double_notify, LIFE.signal_wait              | ✓      |
| 11  | connection/control                       | tunnel-connection    | TRANS.control_stream_registration                        | ✓      |
| 12  | connection/protocol                      | tunnel-connection    | TRANS.protocol_selector_new, TRANS.protocol_auto_refresh | ✓      |
| 13  | tunnelrpc/pogs/registration_server       | tunnel-rpc           | REG.capnp_*, REG.registration_rpc_*                      | ✓      |
| 14  | tunnelrpc/registration_client            | tunnel-rpc           | REG.connect_request_roundtrip, REG.udp_session_*         | ✓      |
| 15  | tunnelrpc/registration_server            | tunnel-rpc           | REG.manage_configuration_rpc                             | ✓      |
| 16  | supervisor/supervisor                    | tunnel-supervisor    | LIFE.protocol_fallback_state_machine                     | ✓      |
| 17  | supervisor/tunnel                        | tunnel-supervisor    | LIFE.protocol_fallback_state_machine                     | ✓      |
| 18  | management/service                       | tunnel-management    | MGMT.disable_diagnostic_routes, MGMT.read_events_*       | ✓      |
| 19  | ingress/config                           | tunnel-ingress-proxy | PROXY.single_origin_sets_config                          | ✓      |
| 20  | ingress/ingress                          | tunnel-ingress-proxy | PROXY.parse_ingress_table_driven                         | ✓      |
| 21  | ingress/origin_connection                | tunnel-ingress-proxy | PROXY.stream_tcp_bidirectional                           | ✓      |
| 22  | ingress/origin_dialer                    | tunnel-ingress-proxy | PROXY.raw_tcp_service_connect_fail                       | ✓      |
| 23  | ingress/origin_proxy                     | tunnel-ingress-proxy | PROXY.tcp_over_ws_service_connect                        | ✓      |
| 24  | ingress/origin_service                   | tunnel-ingress-proxy | PROXY.single_origin_services                             | ✓      |
| 25  | ingress/rule                             | tunnel-ingress-proxy | PROXY.rule_matches_12_cases                              | ✓      |
| 26  | proxy/proxy                              | tunnel-ingress-proxy | PROXY.single_origin_http + 13 more                       | ✓      |
| 27  | edgediscovery/allregions/address         | tunnel-transport     | TRANS.addr_used_by                                       | ✓      |
| 28  | edgediscovery/allregions/discovery       | tunnel-transport     | TRANS.region_discovery_dns                               | ✓      |
| 29  | edgediscovery/allregions/region          | tunnel-transport     | TRANS.region_init_modes                                  | ✓      |
| 30  | edgediscovery/allregions/regions         | tunnel-transport     | TRANS.regions_pool_management                            | ✓      |
| 31  | edgediscovery/allregions/usedby          | tunnel-transport     | TRANS.addr_used_by                                       | ✓      |
| 32  | edgediscovery/dial                       | tunnel-transport     | (transitive: edge discovery)                             | ✓      |
| 33  | edgediscovery/edgediscovery              | tunnel-transport     | TRANS.edge_give_back + 7 more                            | ✓      |

**Result: 33/33 Must atoms covered.**

Atoms 1–4 are covered by placeholder contracts derived from cmd/* test
file names. These placeholders will be refined during S3 implementation
when the actual Go test code is read.

### Invariant Coverage (29 invariants)

All 29 invariants from [invariants](invariants.md) are covered:

| Invariant | Covered by Contracts                                                    | Source  |
| --------- | ----------------------------------------------------------------------- | ------- |
| ARCH-1    | TRANS.quic_server_* (4 proxy-type dispatch)                             | Batch C |
| ARCH-2    | ADR-018 supplementary contract (compile-time)                           | Supp.   |
| ARCH-3    | ADR-012 supplementary contract (compile-time)                           | Supp.   |
| ARCH-4    | TRANS.quic_server_* (enum dispatch)                                     | Batch C |
| ARCH-5    | Rust-specific; S3 design contract                                       | Supp.   |
| ARCH-6    | SESSION.serve_parent_ctx_canceled, ADR-019                              | Batch C |
| TRANS-1   | TRANS.quic_server_*, TRANS.control_stream_registration                  | Batch C |
| TRANS-2   | TRANS.protocol_selector_new, TRANS.protocol_auto_refresh                | Batch C |
| TRANS-3   | TRANS.edge_no_addrs_left, TRANS.edge_only_one_left                      | Batch D |
| REG-1     | REG.registration_rpc_success                                            | Batch C |
| PROXY-1   | PROXY.parse_ingress_table_driven                                        | Batch D |
| PROXY-2   | PROXY.find_matching_rule, PROXY.rule_matches_12_cases                   | Batch D |
| PROXY-3   | CONFIG.update_configuration, CONFIG.override_warp_routing               | Batch D |
| PROXY-4   | SESSION.pipe_close_* (4 contracts)                                      | Batch C |
| SESSION-1 | SESSION.serve_idle_timeout, SESSION.legacy_close_idle                   | Batch C |
| SESSION-2 | SESSION.close_idempotent                                                | Batch C |
| SESSION-3 | SESSION.serve_migrate, SESSION.muxer_migrate                            | Batch C |
| SESSION-4 | SESSION.muxer_register_twice                                            | Batch C |
| CONFIG-1  | ADR-013 obligation (exact round-trip)                                   | Batch B |
| CONFIG-2  | CONFIG.update_configuration, CONFIG.concurrent_update_and_read          | Batch D |
| CONFIG-3  | CONFIG.yaml_file_settings                                               | Batch D |
| CRED-1    | CRED.credentials_read, CRED.find_origin_cert_valid                      | Batch D |
| MGMT-1    | MGMT.start_stream_different_actor, MGMT.start_stream_same_actor         | Batch D |
| ERR-1     | FUZZ.* (8 contracts), LIFE.signal_double_notify                         | C+D     |
| ERR-2     | PROXY.error_propagation_502 + invariant proptest stub                   | Batch D |
| ERR-3     | ERR.backoff_retries, ERR.backoff_grace_period, ERR.backoff_max_duration | Batch D |
| LIFE-1    | LIFE.manager_* (3 contracts), ADR-014 obligation                        | Batch D |
| LIFE-2    | ADR-015 → supervisor shutdown contracts                                 | Supp.   |
| LIFE-3    | LIFE.protocol_fallback_state_machine                                    | Batch D |

**Result: 29/29 invariants covered.**

4 supplementary contracts (ARCH-2, ARCH-3, ARCH-5, LIFE-2) are
Rust-specific — they have no direct Go test equivalent. Their coverage
comes from compile-time checks and property tests generated during S3.
They are NOT counted in the 327 total.

### ADR Parity Obligation Coverage (6 ADRs)

| ADR     | Mitigating Contracts                                                                                                                                     | Status |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| ADR-005 | ERR.backoff_retries, ERR.backoff_grace_period, ERR.backoff_max_duration, ERR.backoff_max_duration_retries, ERR.backoff_retry_forever, ERR.backoff_cancel | ✓      |
| ADR-013 | CONFIG.origin_request_roundtrip + dedicated round-trip contract                                                                                          | ✓      |
| ADR-014 | LIFE.manager_add_remove, LIFE.manager_duplicate, LIFE.manager_error_channel                                                                              | ✓      |
| ADR-017 | CRED.redirect_*, CRED.jwt_*, MGMT.access_token_query_middleware, PROXY.carrier_is_access_response, PROXY.access_validator_*                              | ✓      |
| ADR-018 | ARCH-2 supplementary contract (compile-time)                                                                                                             | ✓      |
| ADR-019 | SESSION.serve_parent_ctx_canceled, SESSION.serve_idle_timeout, SESSION.serve_read_errors                                                                 | ✓      |

**Result: 6/6 ADR obligations covered.**

### Parity-Specific Risk Mitigation (6 risks)

| Risk | Description                            | Mitigating Contracts                                                                            | Status |
| ---- | -------------------------------------- | ----------------------------------------------------------------------------------------------- | ------ |
| R1.5 | defer/panic teardown parity            | CONFIG.concurrent_update_and_read, PROXY.ws_conn_no_goroutine_leak, SESSION.muxer_parallel_icmp | ✓      |
| R1.6 | CustomDuration serialization asymmetry | CONFIG.origin_request_roundtrip, ADR-013 obligation                                             | ✓      |
| R2.3 | Channel capacity close/deadlock        | SESSION.muxer_rate_limit, MGMT.session_insert_overflow, TRANS.observer_events_dont_block        | ✓      |
| R5.1 | Protocol fallback deviation            | LIFE.protocol_fallback_state_machine, 6 HTTP/2 skip contracts                                   | ✓      |
| R5.2 | Cap'n Proto framing                    | REG.capnp_connection_options_roundtrip + wire capture design                                    | ✓      |
| R6.6 | Scope boundary leakage                 | 18 skip contracts + 7 Could-tier atom exclusions + Won't exclusion register                     | ✓      |

**Result: 6/6 parity-specific risks mitigated.**

---

## Fuzz Parity Strategy

### Fuzz Target Inventory (8 targets)

| #   | Go Fuzz Target                   | Contract ID                     | Crate                | S3 Stage |
| --- | -------------------------------- | ------------------------------- | -------------------- | -------- |
| 1   | FuzzRegistrationDatagram         | FUZZ.registration_datagram      | tunnel-session       | 3.4      |
| 2   | FuzzPayloadDatagram              | FUZZ.payload_datagram           | tunnel-session       | 3.4      |
| 3   | FuzzRegistrationResponseDatagram | FUZZ.registration_response      | tunnel-session       | 3.4      |
| 4   | FuzzICMPDatagram                 | FUZZ.icmp_datagram              | tunnel-session       | 3.4      |
| 5   | FuzzIPDecoder                    | FUZZ.ip_decoder                 | tunnel-session       | 3.4      |
| 6   | FuzzICMPDecoder                  | FUZZ.icmp_decoder               | tunnel-session       | 3.4      |
| 7   | FuzzNewIdentity                  | FUZZ.cloudflared_tracing_string | common-observability | 3.1      |
| 8   | FuzzNewAccessValidator           | FUZZ.access_validator           | tunnel-ingress-proxy | 3.6      |

### Framework Selection

| Component        | Go                                   | Rust                                                     |
| ---------------- | ------------------------------------ | -------------------------------------------------------- |
| Fuzz engine      | `testing.F` (Go 1.18+)               | `cargo fuzz` (libFuzzer backend)                         |
| Input generation | `testing.F.Add()` seeds + mutation   | `libfuzzer_sys::fuzz_target!` + `arbitrary::Arbitrary`   |
| Corpus storage   | `testdata/fuzz/` in Go source        | `oracle/captures/fuzz/` (shared) + `fuzz/corpus/` (Rust) |
| Structured input | `(*testing.F).Fuzz(func(t, []byte))` | `Arbitrary` derive for structured types                  |
| CI integration   | `go test -fuzz=Fuzz* -fuzztime=30s`  | `cargo fuzz run <target> -- -max_total_time=30`          |

**Decision: `cargo fuzz` (libFuzzer) as primary engine.** Mature,
coverage-guided, OSS-Fuzz compatible. `bolero` not needed — all 8
targets are crash-finding (no-panic property), not stateful property
tests.

### Fuzz Contract TOML Structure

```toml
[[contract]]
id = "FUZZ.registration_datagram"
description = """
Decoding arbitrary bytes as a registration datagram must not panic.
Any valid registration datagram must round-trip through marshal/unmarshal.
"""
category = "fuzz"
parity_level = "semantic"
tier = "must"
status = "red"

[contract.trace]
s1_atoms = ["quic/v3/datagram"]
s1_tests = ["FuzzRegistrationDatagram"]
s2_invariant = ""
s2_risks = []
s2_adrs = []

[contract.oracle]
type = "property"
deterministic = true
timeout_ms = 30000

[contract.comparison]
mode = "no_panic"
```

### Parity Properties

| Property    | Description                                       | All 8 targets?           |
| ----------- | ------------------------------------------------- | ------------------------ |
| No panic    | Arbitrary input never causes panic                | Yes                      |
| No OOB      | Arbitrary input never causes out-of-bounds access | Yes (Rust memory safety) |
| Round-trip  | Valid inputs round-trip through encode→decode     | 6/8                      |
| Equivalence | Go decode == Rust decode for shared corpus        | 6/8                      |

### Corpus Management

#### Phase 1 — Seed Corpus Extraction (S3 prerequisite)

1. Clone cloudflare/cloudflared@2026.3.0
2. Check for committed corpus in `testdata/fuzz/` directories
3. If present: copy to `oracle/captures/fuzz/{TargetName}/`
4. If absent: run Go fuzz targets for 60s each to generate seed corpus
5. Copy generated corpus to `oracle/captures/fuzz/{TargetName}/`
6. Record provenance in `oracle/captures/fuzz/README.md`

#### Phase 2 — Cross-Validation (S4)

```text
                    ┌──────────────────┐
   Go corpus ──────→│  Rust fuzz target │──→ no panic? ✓
   (oracle seeds)   │  (cargo fuzz)     │──→ same result? ✓
                    └──────────────────┘
                              │
                    ┌─────────▼────────┐
   Rust corpus ────→│  Go oracle binary │──→ same result? ✓
   (new findings)   │  (subprocess)     │──→ divergence = parity bug
                    └──────────────────┘
```

### Fuzz Implementation Patterns

**Pattern 1: Decode-Only** (6 targets in tunnel-session)

```rust
#![no_main]
use libfuzzer_sys::fuzz_target;
use tunnel_session::datagram::RegistrationDatagram;

fuzz_target!(|data: &[u8]| {
    // Property: no panic on arbitrary bytes
    let _ = RegistrationDatagram::unmarshal(data);
});
```

**Pattern 2: Structured Round-Trip** (where applicable)

```rust
#![no_main]
use libfuzzer_sys::fuzz_target;
use tunnel_session::datagram::RegistrationDatagram;

fuzz_target!(|data: &[u8]| {
    if let Ok(dg) = RegistrationDatagram::unmarshal(data) {
        let encoded = dg.marshal();
        let decoded = RegistrationDatagram::unmarshal(&encoded)
            .expect("round-trip must succeed");
        assert_eq!(dg, decoded);
    }
});
```

**Pattern 3: String Parsing** (FUZZ.cloudflared_tracing_string)

```rust
#![no_main]
use libfuzzer_sys::fuzz_target;
use common_observability::identity::Identity;

fuzz_target!(|data: &[u8]| {
    if let Ok(s) = std::str::from_utf8(data) {
        let _ = Identity::new(s);
    }
});
```

**Pattern 4: Validation** (FUZZ.access_validator)

```rust
#![no_main]
use libfuzzer_sys::fuzz_target;
use tunnel_ingress_proxy::validation::AccessValidator;

fuzz_target!(|data: &[u8]| {
    if let Ok(s) = std::str::from_utf8(data) {
        let _ = AccessValidator::new(s);
    }
});
```

### CI Integration

| Mode                 | Scope                                           | Time   | Frequency |
| -------------------- | ----------------------------------------------- | ------ | --------- |
| **Per-PR**           | Corpus replay only (`-runs=0`)                  | ~10s   | Every PR  |
| **Nightly**          | Coverage-guided mutation (`-max_total_time=60`) | ~8 min | Daily     |
| **Cross-validation** | Rust corpus → Go oracle comparison              | ~5 min | Weekly    |

---

## S4 Execution Model

### Architecture

S4 is the verification stratum. It executes parity contracts to confirm
that the Rust implementation matches the Go oracle. The execution model
is designed in S2.7 but implemented in S4.

```text
┌─────────────────────────────────────────────────────────┐
│                    S4 Parity Runner                      │
│                                                         │
│  ┌───────────┐  ┌──────────────┐  ┌──────────────────┐ │
│  │ TOML      │  │ Oracle       │  │ Rust Binary      │ │
│  │ Contract  │→ │ Subprocess   │→ │ Under Test       │ │
│  │ Reader    │  │ Runner       │  │ Runner           │ │
│  └───────────┘  └──────────────┘  └──────────────────┘ │
│        │              │                    │            │
│        ▼              ▼                    ▼            │
│  ┌──────────────────────────────────────────────┐      │
│  │              Diff & Report Engine             │      │
│  └──────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────┘
```

### Contract Execution Modes

| Oracle Type           | Execution Path                                | Automated?                        |
| --------------------- | --------------------------------------------- | --------------------------------- |
| **oracle-comparable** | Run Go binary → run Rust binary → diff        | Yes                               |
| **oracle-captured**   | Load pre-recorded Go output → run Rust → diff | Yes                               |
| **behavioral**        | Run Rust test → check assertions pass         | Yes (`cargo test`)                |
| **property**          | Run proptest/fuzz → check invariants hold     | Yes (`cargo test` + `cargo fuzz`) |

**Execution priority in S4:**

1. `behavioral` + `property` contracts (~265 of 327): verified first,
   no oracle binary needed. Standard `cargo test` and `cargo fuzz`.
2. `oracle-captured` contracts (~1 currently): load fixture from
   `oracle/captures/`, compare Rust output. Needs fixtures but not
   oracle binary at runtime.
3. `oracle-comparable` contracts (0 assigned currently): requires oracle
   binary at runtime. Emerge during S4 for CLI and config output.

### Oracle Runner Design

```rust
pub struct OracleRunner {
    binary_path: PathBuf,     // oracle/cloudflared-amd64-linux
    binary_hash: [u8; 32],    // SHA-256 from oracle/README.md
    captures_dir: PathBuf,    // oracle/captures/
    timeout: Duration,        // from TOML contract
}

impl OracleRunner {
    /// Verify binary hash before any execution
    pub fn verify_binary(&self) -> Result<(), OracleError>;

    /// Run oracle with args and env, capture output
    pub fn run(&self, args: &[&str], env: &[(&str, &str)])
        -> Result<OracleOutput, OracleError>;

    /// Load pre-recorded capture file
    pub fn load_capture(&self, path: &str)
        -> Result<Vec<u8>, OracleError>;
}

pub struct OracleOutput {
    pub exit_code: i32,
    pub stdout: Vec<u8>,
    pub stderr: Vec<u8>,
    pub duration: Duration,
}
```

### Comparison Engine

| Mode         | Implementation                                                 | When to Use              |
| ------------ | -------------------------------------------------------------- | ------------------------ |
| `exit_code`  | `assert_eq!(oracle.exit_code, rust.exit_code)`                 | CLI commands             |
| `stdout`     | `assert_eq!(normalize(oracle.stdout), normalize(rust.stdout))` | Text output              |
| `stderr`     | `assert_eq!(normalize(oracle.stderr), normalize(rust.stderr))` | Error messages           |
| `json_path`  | JSONPath extraction → structural comparison                    | JSON API responses       |
| `proto_diff` | Cap'n Proto binary decode → structural comparison              | RPC wire frames          |
| `file_diff`  | File content comparison after both runs                        | Config file output       |
| `regex`      | regex match against Rust output                                | Non-deterministic output |
| `no_panic`   | Process exited normally (fuzz)                                 | Fuzz targets             |
| `custom`     | Test-specific comparison function                              | Special cases            |

### S4 Aggregate Report

```text
S4 Parity Verification Report — 2026-XX-XX
═══════════════════════════════════════════════════════════
Stage 3.0: 6/6 verified (100%)    ████████████████████ ✓
Stage 3.1: 62/64 verified (97%)   ███████████████████░ …
  skip: 11
Stage 3.2: 10/10 verified (100%)  ████████████████████ ✓
...

Summary
───────
Total contracts:      327
  Verified:           309  (95%)
  Green (unverified):   0  (0%)
  Skip:                18  (5%)
  Red:                  0  (0%)

Invariant coverage:   29/29  ✓
Fuzz coverage:         8/8   ✓
ADR obligations:       6/6   ✓
Risk mitigations:      6/6   ✓
```

### S4 Gate Criteria

S4 exit gate requires all of the following:

1. **All Must-tier contracts (157) are `verified`** — oracle comparison
   confirms parity for oracle-comparable/captured contracts;
   `cargo test` passes for behavioral/property contracts.
2. **All Should-tier contracts are `verified` or documented** — any
   remaining `red` contracts require an acceptance note explaining
   the behavioral delta and its impact.
3. **All 8 fuzz targets pass corpus replay** — Go corpus fed through
   Rust fuzz targets with zero panics.
4. **Wire capture comparison passes** — Cap'n Proto frames match between
   Go and Rust for registration, config update, and reconnection flows.
5. **CLI output comparison passes** — oracle-comparable contracts for
   tunnel run, access commands, and config file parsing.
6. **No green-to-red regressions** during S4 execution.
7. **Aggregate report shows >=95% verified** across all tiers.
8. **Access parity section complete** ([ADR-017](adr/017-access-s4-drop-in-contract.md)):
   explicit sign-off on each Should-tier Access atom.

---

## Exit Gate

S2.7 exit gate checklist:

- [x] TOML contract schema covers all fields for S3 lifecycle and S4
      execution
- [x] Oracle interaction model handles 4 comparison modes
- [x] Red-by-default lifecycle has clear state transitions and CI rules
- [x] All 6 ADR parity obligations documented with fixtures
- [x] Scope exclusions complete: Won't (8), Could (18), RR-deferred
      (12), permanent (6)
- [x] Access parity boundary explicit ([ADR-017](adr/017-access-s4-drop-in-contract.md))
- [x] Contract catalog: 327 contracts (157 must + 144 should + 18 skip
      + 8 fuzz)
- [x] All 33 Must-tier atoms have >=1 contract
- [x] All 29 invariants covered (4 supplementary noted)
- [x] All 8 fuzz targets mapped with implementation patterns
- [x] Fuzz framework selected (`cargo fuzz` / libFuzzer)
- [x] S4 execution model defined (oracle runner, comparison engine,
      gate criteria)
- [x] Coverage matrix verified across all 9 verification criteria
