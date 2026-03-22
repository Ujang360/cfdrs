# Agent Policy

This policy applies to all agents operating in this repository:
Claude, Copilot, and Codex. Read before doing anything.

## Stratum Awareness

This repository follows a four-stratum program lifecycle:

```text
S1/RAW       CLOSED    docs/01-rr/s1/    — Go behavior baseline
S2/SUBSTRATE ACTIVE    docs/01-rr/s2/    — Rust-aware design
S3/COOK      PENDING   codebase          — Implementation
S4/SERVING   PENDING   docs/01-rr/s4/    — Verification
```

Always identify which stratum the current task belongs to.
Never perform S3 work (writing Rust code) until S2 exit gate is passed.
Never perform S4 work until S3 exit gate is passed.

## Session Scope

**One deliverable. One session. One concern.**

Scope each session to a single phase deliverable (e.g., one ADR,
one risk register section, one architecture diagram, one catalog
review). Cross-crate reasoning is expected in S2.

Signs of context blowup — stop immediately if you observe these:

- Contradicting decisions made earlier in the session
- Losing track of which S2 phase the current work belongs to
- Making design claims without S1 atom references

Recovery: stop, start a fresh session, re-read only the specific
deliverable being worked on.

## S1 Reference Policy

S1/RAW is the behavioral oracle for all design and implementation
decisions. Before making any behavioral claim, verify it against:

1. The relevant atom in `docs/01-rr/s1/atoms/`
2. The relevant catalog in `docs/01-rr/s1/catalogs/`
3. The hub atom list in `docs/01-rr/s1/audit-analysis.md`

Never infer behavior that is not visible in S1 atom docs.

## Critical Path

These 7 atoms span ≥12 catalogs and are the load-bearing walls.
They drive architecture decisions in S2 phases 2.6 and 2.8:

```text
tunnel/configuration   (14) — central dispatch
connection/control     (14) — RPC control stream
tunnel/cmd             (13) — primary CLI command
connection/protocol    (13) — protocol selection
management/service     (12) — management runtime
quic/v3/session        (12) — QUIC session lifecycle
supervisor/tunnel      (12) — supervisor loop
```

## What You Must Never Do

- Make behavioral claims not traceable to S1 atoms
- Skip the S2 exit gate (phase 2.10 coherency audit)
- Produce S2 deliverables that contradict S1 catalog evidence

## S3 Rules (activate when S3 is ACTIVE)

These rules are dormant while S2 is the active stratum.
They apply once S3/COOK becomes ACTIVE.

### S3 Session Scope

**One crate. One session. One concern.**

If you find yourself touching more than one crate's implementation
in a single session, stop and split the work.

Recovery: stop, start a fresh session, re-read only the specific
file being worked on, continue from last passing `cargo check`.

### S3 Parity Discipline

Parity TOML must be written before implementation. No exceptions.
The Go oracle binary is the ground truth. The parity TOML is the
contract. The Rust implementation must satisfy the contract —
never redefine the contract to match a wrong implementation.

### S3 Prohibitions

- Start S3 implementation without a parity TOML
- Use `unwrap()` or `expect()` in library crates
- Add `unsafe` without a `// SAFETY:` comment
- Break `cargo check --workspace`
- Redefine a parity TOML to match a wrong implementation

## Markdown Rules

- Always use markdown links, except self-referential links
- Always fix markdown lint problems
- Follow `.markdownlint.json` configuration
