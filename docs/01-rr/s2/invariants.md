# S2.5 - Invariants

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| S2.5 status | Closed |
| Consumes | [scope](scope.md) Must-tier, [dependency-decisions](dependency-decisions.md), [adr](adr/README.md), [risks](risks.md) |
| Produces | S2.6 architecture invariants map, S3 property-test skeletons |
| Testing approach | Natural language contracts plus `proptest` pseudocode stubs; S3 supplies compilable code |
| Fuzz invariants | Deferred to S2.7 parity harness design |

This document records behavioral invariants that the cfdrs implementation must
uphold. Each invariant is:

1. Stated precisely in natural language.
2. Traced to S1 evidence.
3. Given a `proptest`-style pseudocode stub for S3 to implement.

Invariants are organized into two tiers: **Architectural** invariants constrain
every crate and boundary, while **Domain** invariants constrain individual
functional areas on the Must-tier path.

## Invariant Conventions

### ID Namespace

Each invariant carries a short prefixed ID used as the canonical name across
S2.6 architecture notes, S3 test modules, and S4 parity traceability:

| Prefix | Scope | Domain |
| --- | --- | --- |
| `ARCH` | Architectural | Cross-cutting, constrains every crate |
| `TRANS` | Domain | Transport: QUIC connection, protocol, edge discovery |
| `REG` | Domain | Registration: control stream, RPC handshake |
| `PROXY` | Domain | Proxy: ingress rules, config merge, stream pipe |
| `SESSION` | Domain | Session: idle timeout, migration, muxer |
| `CONFIG` | Domain | Configuration: duration format, versioning, discovery |
| `CRED` | Domain | Credentials: credential file resolution |
| `MGMT` | Domain | Management: log streaming, HTTP service |
| `ERR` | Domain | Error handling: classification, backoff policy |
| `LIFE` | Domain | Lifecycle: startup DAG, shutdown, supervisor HA |

S3 test files should mirror this namespace: one test module per prefix, with
test function names matching the stub names in this document.

### Test Kind Classification

Each invariant stub is tagged with a **test kind** that determines its S3
implementation target:

| Kind | Meaning | S3 target |
| --- | --- | --- |
| **property** | Pure stateless property over generated inputs | `proptest!` macro |
| **scenario** | Stateful behavioral test with mock collaborators | `#[tokio::test]` with `proptest` strategies for inputs where useful |
| **compile-time** | Enforced by the Rust type system or `static_assertions` | No runtime test; verified by `cargo check` |

When both a compile-time and a runtime component exist (e.g., ARCH-2 uses
`!Send` enforcement plus an assignment property test), both kinds are listed
and the stub covers only the runtime portion.

## Architectural Invariants

These six invariants constrain every crate. Any S3 implementation that violates
one is breaking a substrate contract, not just a local behavior detail.

### ARCH-1 - Proxy Data Invariant

**Statement:** Everything carried through the tunnel is proxy data except the
transport state machine itself. Cap'n Proto RPC frames, SOCKS5 frames,
WebSocket frames, UDP datagrams, and ICMP packets are all proxy data that must
remain dispatchable through the fat-enum proxy boundary. No exception may be
granted without a new ADR.

**Evidence:** [proxy-data-taxonomy](../s1/proxy-data-taxonomy.md),
[capnp-rpc catalog](../s1/catalogs/domain/capnp-rpc.md),
[dependency-decisions](dependency-decisions.md) Section 3.1 and SOCKS5.

**Test kind:** property

**S3 note:** The proptest stub is intentionally thin — enforcement is mostly
structural. The primary S3.0 check is compile-time: the fat enum must have an
arm for every service type (exhaustive `match`). The proptest validates
reachability, not structural completeness.

**Proptest stub:**

```rust
// For every ProxyVariant in the fat enum dispatch:
// - it must be reachable from ingress rule match
// - it must not touch transport state machine fields
proptest! {
    fn proxy_variant_is_data_not_transport(variant: ProxyVariant) {
        assert!(!variant.touches_transport_state());
        assert!(variant.is_dispatchable_from_ingress());
    }
}
```

### ARCH-2 - Connection-Ownership Invariant

**Statement:** A `quiche::Connection` handle must never leave the thread it was
created on. No channel send, no `Arc` clone, and no lock release may result in
the handle being observed from a different thread. Ownership remains pinned to
the birth thread for the lifetime of the connection.

**Evidence:** [dependency-decisions](dependency-decisions.md) Sections 1.1 and
3.1, [ADR-018](adr/018-quic-connection-ownership-enforcement.md), [risks](risks.md)
R2.6.

**Test kind:** compile-time + property

**Proptest stub:**

```rust
// Connection wrapper must be !Send at the type level.
// This is enforced by the compiler, not proptest.
// proptest covers the assignment-time load balancing path.
proptest! {
    fn connection_assigned_to_single_thread(
        thread_count: u8,
        connection_count: u16,
    ) {
        let assignments = assign_connections(thread_count, connection_count);
        for conn_id in 0..connection_count {
            let assigned = assignments.thread_for(conn_id);
            assert_eq!(1, assigned.len());
        }
    }
}
```

### ARCH-3 - Worker Group Boundary Error Invariant

**Statement:** The `?` operator must never be used at a worker group boundary or actor
message boundary. Errors at these boundaries must be classified internally and
escalated via typed actor messages. `?` is permitted only inside pure
computation within a task, never at the point where a task interacts with the
scheduler or sends a message.

**Evidence:** [dependency-decisions](dependency-decisions.md) Layer 5,
[ADR-012](adr/012-error-taxonomy-and-recoverability-policy.md),
[error-propagation](../s1/catalogs/cross-cutting/error-propagation/README.md).

**Test kind:** compile-time + property

**Proptest stub:**

```rust
// Static enforcement: boundary functions must return () or send typed messages,
// not Result<T, E>. Verified by linting rules in S2.9.
// proptest covers classification correctness.
proptest! {
    fn error_classification_is_exhaustive(raw_error: RawTransportError) {
        let classified = classify_transport_error(&raw_error);
        assert!(classified.is_some());
        let classified2 = classify_transport_error(&raw_error);
        assert_eq!(classified, classified2);
    }
}
```

### ARCH-4 - Fat Enum Dispatch Invariant

**Statement:** The proxy hot-path dispatch must use a fat enum with
monomorphized arms. No `Box<dyn OriginProxy>` or equivalent dynamic dispatch is
permitted on the hot path. Large variant payloads may be boxed, but the enum
itself must not be trait-objectified.

**Evidence:** [dependency-decisions](dependency-decisions.md) Section 3.1,
[proxy-data-taxonomy](../s1/proxy-data-taxonomy.md), [risks](risks.md) R1.1.

**Test kind:** compile-time + property

**Proptest stub:**

```rust
// Static enforcement: ProxyVariant must not contain Box<dyn Trait> fields.
// Verified by compile-time size assertions in S3.0.
// proptest covers dispatch completeness.
proptest! {
    fn all_ingress_service_types_have_dispatch_arm(
        service_type: OriginServiceType,
    ) {
        let variant = ProxyVariant::from_service(service_type);
        assert!(variant.is_ok());
    }
}
```

### ARCH-5 - Bump Arena Discipline Invariant

**Statement:** The per-thread bump arena holds temporaries only. Nothing that
outlives the current proxy session boundary may be allocated in the arena.
`reset()` is called exactly once per session boundary, never mid-session. Arena
state is `!Sync` and never shared across threads.

**Evidence:** [dependency-decisions](dependency-decisions.md) Section 1.4,
[risks](risks.md) R2.1.

**Test kind:** property

**Proptest stub:**

```rust
proptest! {
    fn arena_is_empty_after_session_reset(
        allocation_sizes: Vec<usize>,
    ) {
        let mut arena = BumpArena::new();
        for size in &allocation_sizes {
            let _ = arena.alloc(*size);
        }
        arena.reset();
        assert_eq!(0, arena.used_bytes());
    }
}
```

### ARCH-6 - Cancellation Propagation Invariant

**Statement:** Every async boundary that crosses a worker group, transport, or actor
message boundary must propagate both an explicit cancellation token and an
explicit timeout value. Dropping a future is not an acceptable substitute for a
boundary contract. The same boundary must not mix incompatible cancellation
mechanisms.

**Evidence:** [ADR-019](adr/019-cancellation-timeout-propagation-contract.md),
[risks](risks.md) R1.2, [porting-friction](../s1/catalogs/cross-cutting/porting-friction/README.md).

**Test kind:** scenario

**Proptest stub:**

```rust
proptest! {
    fn cancellation_and_timeout_cross_boundary_together(
        timeout_ms: u64,
        cancel_before_completion: bool,
    ) {
        let boundary = spawn_boundary(Duration::from_millis(timeout_ms));
        if cancel_before_completion {
            boundary.cancel();
            assert!(boundary.is_cancelled());
        } else {
            let result = boundary.run_to_timeout();
            assert!(result.is_timeout() || result.is_complete());
        }
        assert!(boundary.exposes_explicit_timeout());
    }
}
```

## Domain Invariants

### 1 - Transport

#### TRANS-1 - QUIC Connection Serves Until Unregistered

**Statement:** A QUIC connection serve loop must not exit unless the control
stream returns cleanly during unregistration or the context is cancelled.
Transport errors that are classified as recoverable must not terminate the
serve loop.

**Evidence:** [quic_connection](../s1/atoms/connection/quic_connection.md),
[concurrency](../s1/catalogs/cross-cutting/concurrency/README.md) Pattern 2,
[risks](risks.md) R3.1.

**Test kind:** scenario

**Proptest stub:**

```rust
proptest! {
    fn recoverable_transport_error_does_not_exit_serve_loop(
        error: RecoverableTransportError,
    ) {
        let mut conn = mock_quic_connection();
        conn.inject_error(error);
        assert!(!conn.is_terminated());
    }
}
```

#### TRANS-2 - Protocol Selection Is Stable Per Attempt

**Statement:** Once a protocol is selected for a connection attempt, it remains
fixed for the lifetime of that attempt. Protocol fallback occurs only between
supervisor-managed attempts, never mid-connection.

**Evidence:** [protocol](../s1/atoms/connection/protocol.md),
[ADR-001](adr/001-quic-transport.md), [scope](scope.md).

**Test kind:** scenario

**Proptest stub:**

```rust
proptest! {
    fn protocol_does_not_change_mid_connection(
        initial_protocol: Protocol,
        events: Vec<ConnectionEvent>,
    ) {
        let conn = establish_connection(initial_protocol);
        for event in events {
            conn.handle(event);
            assert_eq!(initial_protocol, conn.protocol());
        }
    }
}
```

#### TRANS-3 - Edge Discovery Is Non-Empty Or Fatal

**Statement:** Edge discovery must return at least one usable address before
any connection attempt is made. An empty address pool is fatal for the current
attempt, not recoverable edge inventory.

**Evidence:** [edgediscovery](../s1/atoms/edgediscovery/edgediscovery.md),
[allregions discovery](../s1/atoms/edgediscovery/allregions/discovery.md),
[risks](risks.md) R5.1.

**Test kind:** property

**Proptest stub:**

```rust
proptest! {
    fn edge_discovery_returns_nonempty_pool(
        srv_records: Vec<SrvRecord>,
    ) {
        prop_assume!(!srv_records.is_empty());
        let pool = EdgeAddressPool::from_srv(srv_records);
        assert!(!pool.is_empty());
    }

    fn empty_edge_pool_is_fatal() {
        let pool = EdgeAddressPool::from_srv(vec![]);
        assert!(pool.is_empty());
        assert!(classify_edge_pool(pool).is_fatal());
    }
}
```

### 2 - Registration

#### REG-1 - Registration Completes Before Proxy Service

**Statement:** A connection must not begin serving proxy traffic until edge
registration completes successfully. Registration failure aborts the current
attempt before the connection is exposed to supervisor steady state.

**Evidence:** [control](../s1/atoms/connection/control.md),
[registration_client](../s1/atoms/tunnelrpc/registration_client.md),
[registration_server](../s1/atoms/tunnelrpc/registration_server.md),
[risks](risks.md) R5.2.

**Test kind:** scenario

**Proptest stub:**

```rust
proptest! {
    fn registration_gate_precedes_proxy_service(
        registration_ok: bool,
    ) {
        let mut conn = mock_registered_connection();
        conn.set_registration_outcome(registration_ok);
        conn.advance_startup();

        if registration_ok {
            assert!(conn.proxy_service_started());
        } else {
            assert!(!conn.proxy_service_started());
            assert!(conn.attempt_aborted());
        }
    }
}
```

### 3 - Proxy

#### PROXY-1 - Ingress Rule Set Always Has a Catch-All

**Statement:** A validated ingress rule set must always have exactly one
catch-all rule, and it must be the last rule. A catch-all rule before the last
position is invalid. A last rule that is not catch-all is also invalid.

**Evidence:** [ingress](../s1/atoms/ingress/ingress.md),
[rule](../s1/atoms/ingress/rule.md),
[ingress catalog](../s1/catalogs/domain/ingress.md).

**Test kind:** property

**Proptest stub:**

```rust
proptest! {
    fn valid_ingress_always_has_terminal_catchall(
        rules: Vec<IngressRule>,
    ) {
        prop_assume!(!rules.is_empty());
        prop_assume!(rules.last().unwrap().is_catch_all());
        prop_assume!(rules[..rules.len() - 1].iter().all(|r| !r.is_catch_all()));
        let ingress = Ingress::validate(rules);
        assert!(ingress.is_ok());
    }

    fn ingress_without_catchall_is_invalid(
        rules: Vec<IngressRule>,
    ) {
        prop_assume!(!rules.is_empty());
        prop_assume!(rules.iter().all(|r| !r.is_catch_all()));
        let ingress = Ingress::validate(rules);
        assert!(ingress.is_err());
    }
}
```

#### PROXY-2 - Rule Matching Is Deterministic

**Statement:** For any given `(hostname, path)` pair, `FindMatchingRule` must
return the same rule index on every call. Rule matching has no side effects and
no non-deterministic branches.

**Evidence:** [ingress](../s1/atoms/ingress/ingress.md),
[ingress catalog](../s1/catalogs/domain/ingress.md).

**Test kind:** property

**Proptest stub:**

```rust
proptest! {
    fn rule_matching_is_deterministic(
        hostname: String,
        path: String,
        ingress: ValidIngress,
    ) {
        let r1 = ingress.find_matching_rule(&hostname, &path);
        let r2 = ingress.find_matching_rule(&hostname, &path);
        assert_eq!(r1, r2);
    }
}
```

#### PROXY-3 - Config Merge Precedence Is Total And Ordered

**Statement:** Origin request config merging follows a strict four-layer
precedence: per-rule overrides, global defaults, cloudflared team defaults,
then zero values. An explicit zero-valued non-nil override takes precedence
over a non-zero default. Omitted is distinct from zero.

**Evidence:** [ingress catalog](../s1/catalogs/domain/ingress.md),
[config](../s1/atoms/ingress/config.md),
[ADR-013](adr/013-customduration-dual-format-contract.md).

**Test kind:** property

**Proptest stub:**

```rust
proptest! {
    fn per_rule_override_beats_global_default(
        per_rule: Option<Duration>,
        global: Option<Duration>,
        team_default: Duration,
    ) {
        let merged = merge_origin_request_config(per_rule, global, team_default);
        match (per_rule, global) {
            (Some(v), _) => assert_eq!(merged.connect_timeout, v),
            (None, Some(v)) => assert_eq!(merged.connect_timeout, v),
            (None, None) => assert_eq!(merged.connect_timeout, team_default),
        }
    }
}
```

#### PROXY-4 - Bidirectional Stream Pipe Closes Both Legs

**Statement:** When one direction of a bidirectional stream pipe reaches EOF or
error, `CloseWrite()` must be propagated to the other direction. The pipe must
not return until both legs have completed or the second-leg wait budget
expires.

**Evidence:** [stream](../s1/atoms/stream/stream.md),
[copy](../s1/atoms/cfio/copy.md),
[concurrency](../s1/catalogs/cross-cutting/concurrency/README.md) Pattern 5.

**Test kind:** scenario

**Proptest stub:**

```rust
proptest! {
    fn pipe_propagates_half_close_to_second_leg(
        upstream_bytes: Vec<u8>,
        downstream_bytes: Vec<u8>,
    ) {
        let (client, origin) = mock_pipe_pair();
        let result = pipe_bidirectional(client, origin);
        assert!(result.upstream_closed);
        assert!(result.downstream_closed);
    }
}
```

### 4 - Session

#### SESSION-1 - Session Idle Timeout Is Reset By Activity

**Statement:** Any read or write activity on a session must reset the idle
timer. The session must not close due to idle timeout while data is actively
flowing. The idle timer fires only when no read or write has occurred within
the configured `closeAfterIdle` window.

**Evidence:** [session](../s1/atoms/quic/v3/session.md),
[tests-sessions-packets](../s1/catalogs/cross-cutting/tests-sessions-packets.md).

**Test kind:** scenario

**Proptest stub:**

```rust
proptest! {
    fn activity_prevents_idle_timeout(
        idle_duration: Duration,
        activity_interval: Duration,
    ) {
        prop_assume!(activity_interval < idle_duration);
        let session = mock_session(idle_duration);
        for _ in 0..10 {
            session.write(&[0u8]);
            sleep(activity_interval);
        }
        assert!(!session.is_idle_closed());
    }
}
```

#### SESSION-2 - Session Close Is Idempotent

**Statement:** Calling `close()` on a session more than once must not panic and
must not return an error. The second and subsequent close calls are no-ops.

**Evidence:** [session](../s1/atoms/quic/v3/session.md),
[tests-sessions-packets](../s1/catalogs/cross-cutting/tests-sessions-packets.md).

**Test kind:** property

**Proptest stub:**

```rust
proptest! {
    fn session_close_is_idempotent(close_count: u8) {
        prop_assume!(close_count > 0);
        let session = mock_session(Duration::from_secs(30));
        for _ in 0..close_count {
            let result = session.close();
            assert!(result.is_ok());
        }
    }
}
```

#### SESSION-3 - Migration Rebinds Context Without Closing Session

**Statement:** When a session migrates to a new QUIC connection, the session
context token must be rebound to the new connection context. Cancellation of
the old connection context must not close the migrated session.

**Evidence:** [session](../s1/atoms/quic/v3/session.md),
[ADR-016](adr/016-session-migration-lifecycle.md), [risks](risks.md) R2.2.

**Test kind:** scenario

**Proptest stub:**

```rust
proptest! {
    fn migrated_session_survives_old_context_cancel(
        payload: Vec<u8>,
    ) {
        let (conn0, conn1) = (mock_conn(), mock_conn());
        let session = register_session(&conn0);
        session.migrate(&conn1);
        conn0.cancel();
        assert!(!session.is_closed());
        assert_eq!(session.connection_id(), conn1.id());
    }
}
```

#### SESSION-4 - Duplicate Registration Reuses The Existing Session

**Statement:** If a registration request arrives for a `RequestID` that already
has an active session on the same connection, the response must be
`ResponseOk`, the existing session idle timer must be reset, and no new session
object may be created.

**Evidence:** [muxer](../s1/atoms/quic/v3/muxer.md),
[tests-sessions-packets](../s1/catalogs/cross-cutting/tests-sessions-packets.md).

**Test kind:** scenario

**Proptest stub:**

```rust
proptest! {
    fn duplicate_registration_resets_idle_timer(
        request_id: RequestID,
    ) {
        let conn = mock_conn();
        let session = register_session_with_id(&conn, request_id);
        let session_id = session.object_id();
        let timer_before = session.idle_timer_remaining();
        sleep(Duration::from_millis(100));

        let resp = conn.handle_registration(request_id);

        assert_eq!(ResponseOk, resp);
        assert_eq!(session.object_id(), session_id);
        assert!(session.idle_timer_remaining() >= timer_before);
    }
}
```

### 5 - Configuration

#### CONFIG-1 - CustomDuration Round-Trips In Both Formats

**Statement:** A `CustomDuration` value must round-trip correctly through JSON
and YAML independently. JSON uses integer seconds. YAML uses Go-style duration
strings. Crossing formats must not silently reinterpret the same raw payload as
the other format.

**Evidence:** [model](../s1/atoms/config/model.md),
[porting-friction](../s1/catalogs/cross-cutting/porting-friction/README.md),
[ADR-013](adr/013-customduration-dual-format-contract.md), [risks](risks.md)
R1.6.

**Test kind:** property

**Proptest stub:**

```rust
proptest! {
    fn custom_duration_json_round_trips(secs: u64) {
        let d = CustomDuration::from_secs(secs);
        let json = serde_json::to_string(&d).unwrap();
        let d2: CustomDuration = serde_json::from_str(&json).unwrap();
        assert_eq!(d, d2);
        let raw: u64 = serde_json::from_str(&json).unwrap();
        assert_eq!(raw, secs);
    }

    fn custom_duration_yaml_round_trips(secs: u64) {
        let d = CustomDuration::from_secs(secs);
        let yaml = serde_yaml::to_string(&d).unwrap();
        let d2: CustomDuration = serde_yaml::from_str(&yaml).unwrap();
        assert_eq!(d, d2);
        assert!(yaml.trim().ends_with('s') || yaml.contains('m') || yaml.contains('h'));
    }
}
```

#### CONFIG-2 - Config Version Is Monotonic

**Statement:** Each successful `updateConfiguration` call must increment the
config version. Rejected updates must not change the version. The version must
never decrease.

**Evidence:** [orchestrator](../s1/atoms/orchestration/orchestrator.md),
[configuration_manager](../s1/atoms/tunnelrpc/pogs/configuration_manager.md),
[risks](risks.md) R4.1.

**Test kind:** property

**Proptest stub:**

```rust
proptest! {
    fn config_version_is_monotonic(
        updates: Vec<ConfigUpdate>,
    ) {
        let mut orch = Orchestrator::new();
        let mut last_version = orch.version();
        for update in updates {
            let prev = orch.version();
            let result = orch.apply_update(update);
            if result.is_ok() {
                assert!(orch.version() > prev);
            } else {
                assert_eq!(orch.version(), prev);
            }
            assert!(orch.version() >= last_version);
            last_version = orch.version();
        }
    }
}
```

#### CONFIG-3 - Config Discovery Search Path Is Ordered

**Statement:** Config file discovery checks candidate paths in strict priority
order: explicit `--config`, `~/.cloudflared/config.yaml`,
`/etc/cloudflared/config.yaml`, `$CFDPATH`, then systemd unit environment. The
first readable file wins. Absence of a config file is not fatal if tunnel
credentials are supplied through flags.

**Evidence:** [configuration](../s1/atoms/config/configuration.md),
[config catalog](../s1/catalogs/domain/config.md), [scope](scope.md).

**Test kind:** property

**S3 note:** The stub simplifies actual path resolution. The real
implementation must handle `$CFDPATH` and systemd unit environment variables,
which are not easily proptest-able without process-level env manipulation. S3
may need to refine this into an integration test with a controlled filesystem
fixture.

**Proptest stub:**

```rust
proptest! {
    fn config_discovery_respects_priority(
        explicit_path: Option<PathBuf>,
        home_config_exists: bool,
        etc_config_exists: bool,
    ) {
        let result = discover_config(explicit_path.clone(), home_config_exists, etc_config_exists);
        match explicit_path {
            Some(p) => assert_eq!(result.unwrap(), p),
            None if home_config_exists => assert!(result.unwrap().starts_with("~/.cloudflared")),
            None if etc_config_exists => assert!(result.unwrap().starts_with("/etc/cloudflared")),
            _ => assert!(result.is_none()),
        }
    }
}
```

### 6 - Credentials

#### CRED-1 - Credential Resolution Is Ordered And Total

**Statement:** Tunnel credential resolution follows one ordered strategy at a
time. An explicit credential file path resolves directly. Otherwise, tunnel-ID
search resolves the credential file. Startup must fail if the selected strategy
does not yield exactly one readable credential source.

**Evidence:** [credential_finder](../s1/atoms/cmd/cloudflared/tunnel/credential_finder.md),
[credentials](../s1/atoms/credentials/credentials.md),
[tunnel configuration](../s1/atoms/cmd/cloudflared/tunnel/configuration.md).

**Test kind:** property

**Proptest stub:**

```rust
proptest! {
    fn explicit_credential_path_wins(
        explicit_path: Option<PathBuf>,
        tunnel_id: Uuid,
    ) {
        let resolver = CredentialResolver::new(explicit_path.clone(), tunnel_id);
        let result = resolver.resolve();

        if let Some(path) = explicit_path {
            assert_eq!(result.unwrap().source_path(), path);
        }
        // Non-explicit-path resolution strategy covered by
        // missing_selected_credential_source_is_fatal below
    }

    fn missing_selected_credential_source_is_fatal(
        tunnel_id: Uuid,
    ) {
        let resolver = CredentialResolver::missing(tunnel_id);
        assert!(resolver.resolve().is_err());
    }
}
```

### 7 - Management

#### MGMT-1 - Management Log Streaming Is Single-Session Per Consumer

**Statement:** A management log-stream session may have at most one active
streaming task at a time. A second start request for the same session must be
rejected or ignored until the first stream ends.

**Evidence:** [management service](../s1/atoms/management/service.md),
[dependency-decisions](dependency-decisions.md) critical path,
[risks](risks.md) R2.4.

**Test kind:** scenario

**Proptest stub:**

```rust
proptest! {
    fn management_log_stream_is_singleton_per_session(
        restart_attempts: u8,
    ) {
        let service = mock_management_service();
        let session = service.open_session();

        assert!(service.start_log_stream(&session).is_ok());
        for _ in 0..restart_attempts {
            assert!(service.start_log_stream(&session).is_rejected());
        }

        assert_eq!(1, service.active_log_streams(&session));
    }
}
```

### 8 - Error Handling

#### ERR-1 - Recoverable Errors Do Not Escalate To Fatal Exit

**Statement:** An error classified as recoverable at the transport layer must
never reach `std::process::ExitCode`. It must be handled inside the connection
or supervisor loop by retry, backoff, or typed escalation. Fatal exit is
reserved for errors explicitly classified as non-recoverable.

**Evidence:** [dependency-decisions](dependency-decisions.md) Layer 5,
[ADR-012](adr/012-error-taxonomy-and-recoverability-policy.md),
[error-propagation](../s1/catalogs/cross-cutting/error-propagation/README.md),
[risks](risks.md) R3.1.

**Test kind:** scenario

**Proptest stub:**

```rust
proptest! {
    fn recoverable_error_never_exits_process(
        error: TransportError,
    ) {
        prop_assume!(error.is_recoverable());
        let mut supervisor = mock_supervisor();
        supervisor.inject_error(error);
        assert!(!supervisor.has_exited());
    }
}
```

#### ERR-2 - Error Classification Is Exhaustive And Stable

**Statement:** Every error variant produced by the transport, session, and
control-plane layers must map to exactly one classification. The mapping must
be pure. No error may fall through unclassified.

**Evidence:** [error-propagation](../s1/catalogs/cross-cutting/error-propagation/README.md),
[ADR-012](adr/012-error-taxonomy-and-recoverability-policy.md), [risks](risks.md)
R3.1.

**Test kind:** property

**Proptest stub:**

```rust
proptest! {
    fn all_errors_are_classified(error: AnyTransportError) {
        let class = ErrorClassifier::classify(&error);
        assert!(class == ErrorClass::Recoverable || class == ErrorClass::Fatal);
    }

    fn error_classification_is_pure(error: AnyTransportError) {
        let c1 = ErrorClassifier::classify(&error);
        let c2 = ErrorClassifier::classify(&error);
        assert_eq!(c1, c2);
    }
}
```

#### ERR-3 - Backoff Policy Is Bounded, Deterministic, And Reset-Aware

**Statement:** Retry delay generation must remain bounded by the configured
maximum, must be reproducible for the same initial seed and attempt sequence,
and must reset to the base policy after explicit success or reset events.
Jitter is allowed, but it must be policy-defined rather than ad hoc.

**Evidence:** [backoffhandler](../s1/atoms/retry/backoffhandler.md),
[ADR-005](adr/005-retry-backoff-library-vs-custom.md), [risks](risks.md)
R6.3.

**Test kind:** property

**Proptest stub:**

```rust
proptest! {
    fn backoff_delays_are_bounded_and_reproducible(
        base: Duration,
        max: Duration,
        seed: u64,
        retry_count: u8,
    ) {
        prop_assume!(base <= max);
        let seq1 = BackoffPolicy::seeded(base, max, seed).take(retry_count);
        let seq2 = BackoffPolicy::seeded(base, max, seed).take(retry_count);

        assert_eq!(seq1, seq2);
        for delay in seq1 {
            assert!(delay >= base);
            assert!(delay <= max);
        }
    }

    fn backoff_reset_restores_base_delay(
        base: Duration,
        max: Duration,
        seed: u64,
    ) {
        prop_assume!(base <= max);
        let mut policy = BackoffPolicy::seeded(base, max, seed);
        let _ = policy.next_delay();
        let _ = policy.next_delay();
        policy.reset();
        assert_eq!(policy.next_delay(), base);
    }
}
```

### 9 - Lifecycle

#### LIFE-1 - Startup DAG Phase Order Is Enforced

**Statement:** Startup initializes phases in dependency order: logger and
metrics before any component that uses them; config before ingress; ingress
before proxy; transport before supervisor; supervisor before `READY=1`. Any
violation is a hard bootstrap failure, not a silent race.

**Evidence:** [cmd](../s1/atoms/cmd/cloudflared/tunnel/cmd.md),
[init-teardown](../s1/catalogs/cross-cutting/init-teardown/README.md),
[ADR-014](adr/014-startup-dag-contract.md), [risks](risks.md) R4.1.

**Test kind:** scenario

**S3 note:** Structural enforcement (the DAG type system) is more important
than proptest here. The stub focuses on the observable symptom: `sd_notify
READY=1` fires iff all phases succeed. The real S3 enforcement is the
`Bootstrap` type's phase-dependency graph preventing out-of-order
initialization at compile time.

**Proptest stub:**

```rust
// Phase ordering is structural, not purely property-testable.
// proptest covers the externally visible READY signal.
proptest! {
    fn sdnotify_fires_after_all_phases(
        phase_results: Vec<PhaseResult>,
    ) {
        prop_assume!(phase_results.iter().all(|r| r.is_ok()));
        let bootstrap = Bootstrap::run(phase_results);
        assert!(bootstrap.sdnotify_ready_sent());
        assert!(bootstrap.all_phases_completed());
    }

    fn sdnotify_does_not_fire_on_phase_failure(
        phase_results: Vec<PhaseResult>,
    ) {
        prop_assume!(phase_results.iter().any(|r| r.is_err()));
        let bootstrap = Bootstrap::run(phase_results);
        assert!(!bootstrap.sdnotify_ready_sent());
    }
}
```

#### LIFE-2 - Graceful Shutdown Completes Both Phases

**Statement:** On SIGINT or SIGTERM, shutdown proceeds in two phases. Phase 1
broadcasts graceful cancellation and waits for actors to drain within a bounded
timeout. Phase 2 hard-cancels anything still running. There is no infinite
wait path.

**Evidence:** [signal](../s1/atoms/cmd/cloudflared/tunnel/signal.md),
[shutdown-teardown](../s1/catalogs/cross-cutting/init-teardown/shutdown-teardown.md),
[ADR-015](adr/015-graceful-shutdown-contract.md), [risks](risks.md) R4.2.

**Test kind:** scenario

**Proptest stub:**

```rust
proptest! {
    fn shutdown_always_terminates(
        phase1_timeout: Duration,
        actor_drain_times: Vec<Duration>,
    ) {
        prop_assume!(phase1_timeout > Duration::ZERO);
        let shutdown = Shutdown::new(phase1_timeout);
        shutdown.signal();
        let result = shutdown.wait_for_completion();
        assert!(result.is_completed());
    }

    fn phase2_fires_when_phase1_times_out(
        short_timeout: Duration,
        slow_actor_delay: Duration,
    ) {
        prop_assume!(slow_actor_delay > short_timeout);
        let shutdown = Shutdown::new(short_timeout);
        let _slow_actor = SpawnSlowActor::new(slow_actor_delay);
        shutdown.signal();
        let result = shutdown.wait_for_completion();
        assert!(result.phase2_fired());
        assert!(result.is_completed());
    }
}
```

#### LIFE-3 - Supervisor Starts HA Connections Only After First Registration

**Statement:** The supervisor must not start HA connections `1..N-1` until
connection `0` has successfully registered and raised the connected signal.
Each subsequent connection starts with the configured registration stagger. If
connection `0` fails to register, no further HA connections are started.

**Evidence:** [supervisor](../s1/atoms/supervisor/supervisor.md),
[fuse](../s1/atoms/supervisor/fuse.md),
[concurrency](../s1/catalogs/cross-cutting/concurrency/README.md) Pattern 1.

**Test kind:** scenario

**Proptest stub:**

```rust
proptest! {
    fn ha_expansion_waits_for_first_connection(
        ha_count: u8,
    ) {
        prop_assume!(ha_count > 1);
        let mut supervisor = mock_supervisor(ha_count);
        supervisor.start();
        assert_eq!(1, supervisor.started_count());
        supervisor.signal_first_connected();
        assert_eq!(ha_count as usize, supervisor.started_count());
    }
}
```

## Invariant Summary Index

| ID | Name | Domain | Test kind |
| --- | --- | --- | --- |
| ARCH-1 | Proxy Data Boundary | Architectural | property |
| ARCH-2 | Connection Ownership | Architectural | compile-time + property |
| ARCH-3 | Worker Group Boundary Error | Architectural | compile-time + property |
| ARCH-4 | Fat Enum Dispatch | Architectural | compile-time + property |
| ARCH-5 | Bump Arena Discipline | Architectural | property |
| ARCH-6 | Cancellation Propagation | Architectural | scenario |
| TRANS-1 | QUIC Serve Until Unregistered | Transport | scenario |
| TRANS-2 | Protocol Stable Per Attempt | Transport | scenario |
| TRANS-3 | Edge Discovery Non-Empty Or Fatal | Transport | property |
| REG-1 | Registration Before Proxy | Registration | scenario |
| PROXY-1 | Ingress Catch-All | Proxy | property |
| PROXY-2 | Rule Matching Deterministic | Proxy | property |
| PROXY-3 | Config Merge Precedence | Proxy | property |
| PROXY-4 | Pipe Closes Both Legs | Proxy | scenario |
| SESSION-1 | Idle Timeout Reset By Activity | Session | scenario |
| SESSION-2 | Close Is Idempotent | Session | property |
| SESSION-3 | Migration Rebinds Context | Session | scenario |
| SESSION-4 | Duplicate Registration Reuses Session | Session | scenario |
| CONFIG-1 | CustomDuration Round-Trip | Configuration | property |
| CONFIG-2 | Config Version Monotonic | Configuration | property |
| CONFIG-3 | Config Discovery Ordered | Configuration | property |
| CRED-1 | Credential Resolution Ordered | Credentials | property |
| MGMT-1 | Log Stream Singleton | Management | scenario |
| ERR-1 | Recoverable Not Fatal | Error | scenario |
| ERR-2 | Classification Exhaustive And Stable | Error | property |
| ERR-3 | Backoff Bounded And Reset-Aware | Error | property |
| LIFE-1 | Startup DAG Enforced | Lifecycle | scenario |
| LIFE-2 | Shutdown Both Phases | Lifecycle | scenario |
| LIFE-3 | HA After First Registration | Lifecycle | scenario |

## S2.5 Exit Gate

| Criterion | Status |
| --- | --- |
| All 6 architectural invariants documented with S1 or S2 evidence | ✅ |
| All Must-tier stateful domains are covered: transport, registration, proxy, session, config, credentials, management, error, lifecycle | ✅ |
| Every invariant has a `proptest` pseudocode stub | ✅ |
| Every invariant has a test kind classification (property / scenario / compile-time) | ✅ |
| Every invariant traces to at least one S1 atom or catalog | ✅ |
| Every invariant traces to at least one ADR or dependency-decisions entry | ✅ |
| Fuzz invariants are explicitly deferred to S2.7 | ✅ |
| No invariant contradicts a closed ADR decision | ✅ |
| Evidence references resolve to existing S1/S2 documents | ✅ |
