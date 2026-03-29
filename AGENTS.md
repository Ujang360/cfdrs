# Agent Policy

This policy applies to all agents operating in this repository:
Claude Code, GitHub Copilot, and OpenAI Codex.
Read before doing anything.

Each agent discovers this file through its own entry point:

| Agent           | Entry point                                                                      | Mechanism           |
|-----------------|----------------------------------------------------------------------------------|---------------------|
| Claude Code     | [CLAUDE.md](CLAUDE.md)                                                           | Auto-loaded by CLI  |
| GitHub Copilot  | [.github/copilot-instructions.md](.github/copilot-instructions.md)               | VS Code integration |
| OpenAI Codex    | AGENTS.md (this file)                                                            | Auto-loaded by CLI  |

All three entry points converge here. Do not duplicate rules elsewhere.

## Stratum Awareness

This repository follows a four-stratum program lifecycle:

```text
S1/RAW       CLOSED    docs/01-rr/s1/    — Go behavior baseline
S2/SUBSTRATE ACTIVE    docs/01-rr/s2/    — Rust-aware design (phase 2.7)
S3/COOK      PENDING   codebase          — Implementation
S4/SERVING   PENDING   docs/01-rr/s4/    — Verification
```

Always identify which stratum the current task belongs to.
Never perform S3 work (writing Rust code) until S2 exit gate is passed.
Never perform S4 work until S3 exit gate is passed.

### Stratum Isolation Principle

Each stratum distills the one before it. Once a stratum is
complete, downstream work reads only that stratum — never the
raw source above it.

| Active stratum | Primary reads         | S1 access              |
|----------------|-----------------------|------------------------|
| S2 (current)   | S1 targeted + S2 docs | Allowed for gap-fill   |
| S3             | S2 docs only          | **Exceptional only**   |
| S4             | S2 + S3 artifacts     | **Exceptional only**   |

**Why this matters:** S1 is ~415K tokens (lethal for every agent).
S2 is ~45K tokens total (fits comfortably). The entire purpose of
S2/SUBSTRATE is to distill S1 into a self-sufficient design layer
so that S3 implementation never needs to touch S1.

**When S1 access is exceptional (S3/S4):** Only read S1 when a
specific incoherence or doubt arises that S2 cannot resolve. Log
the gap, fix S2 to cover it, then continue from S2. Never leave
S1 knowledge un-distilled — if you read S1, update S2 to capture
what was missing.

**When S2 is missing something:** The fix is always to add the
missing information to S2 — never to bypass S2 and work from S1
directly. This keeps S2 as the single source of truth for S3.

## Session Scope

**One deliverable. One session. One concern.**

Scope each session to a single phase deliverable (e.g., one ADR,
one risk register section, one architecture diagram, one catalog
review). Cross-crate reasoning is expected in S2.

Signs of context blowup — stop immediately if you observe these:

- Contradicting decisions made earlier in the session
- Losing track of which stratum/phase the current work belongs to
- Reaching for S1 docs during S3 work (read S2 instead)
- Reading more than 3 S2 docs in one session

Recovery: stop, start a fresh session, re-read only the specific
deliverable being worked on.

## Context Window Budget

Total repository documentation is ~530K tokens — no agent can fit
it all. Respect these budgets to avoid context blowup.

### Agent context limits

| Agent          | Context window | Usable budget* |
|----------------|----------------|----------------|
| Claude Code    | ~200K tokens   | ~120K tokens   |
| GitHub Copilot | ~128K tokens   | ~80K tokens    |
| OpenAI Codex   | ~200K tokens   | ~120K tokens   |

*Usable budget = context window minus system prompt, tool
definitions, and conversation history overhead (~40%).

### Document sizes (measured)

| Zone | ~Tokens | Risk |
| --- | --- | --- |
| S1 atoms (`docs/01-rr/s1/atoms/`) | ~195K | **Lethal** |
| S1 catalogs (`docs/01-rr/s1/catalogs/`) | ~220K | **Lethal** |
| [dependency-shopping-cart.md](docs/01-rr/s2/dependency-shopping-cart.md) | ~33K | High |
| S1 root docs (audit-analysis, atom-index, etc.) | ~26K | Moderate |
| [architecture.md](docs/01-rr/s2/architecture.md) | ~13K | Moderate |
| [scope.md](docs/01-rr/s2/scope.md) | ~12K | Moderate |
| [invariants.md](docs/01-rr/s2/invariants.md) | ~9K | Low |
| [dependency-decisions.md](docs/01-rr/s2/dependency-decisions.md) | ~6K | Low |
| [risks.md](docs/01-rr/s2/risks.md) | ~4K | Low |
| Single ADR | ~500 | Negligible |
| AGENTS.md | ~1K | Negligible |

### Mandatory guardrails

1. **Never bulk-read S1 atoms or catalogs.** Use targeted search
   (grep/semantic) to find the specific atom, then read only that
   file. Each atom averages ~600 tokens.
2. **During S3/S4: read S2, not S1.** S2 is the self-sufficient
   design layer. Only fall back to S1 for exceptional gaps, and
   update S2 immediately after.
3. **Never read `dependency-shopping-cart.md` in full.** It is a
   research artifact. Use
   [dependency-decisions.md](docs/01-rr/s2/dependency-decisions.md)
   as the authoritative summary (~6K tokens).
4. **Cap S2 reads to ≤3 docs per session.** If you need more than
   architecture + scope + one ADR, you are likely violating
   "one deliverable, one session."
5. **Prefer search over read.** When verifying a claim, grep for the
   keyword instead of reading the entire document.
6. **Budget check before large reads.** Before opening any file
   >10K tokens, confirm the read is necessary for the current
   deliverable.

## S1 Reference Policy

S1/RAW is the behavioral oracle — but access depends on the
active stratum.

### During S2 (current)

S1 is the primary source. Before making any behavioral claim,
verify it against:

1. The relevant atom in `docs/01-rr/s1/atoms/`
2. The relevant catalog in `docs/01-rr/s1/catalogs/`
3. The hub atom list in
   [audit-analysis.md](docs/01-rr/s1/audit-analysis.md)

Never infer behavior that is not visible in S1 atom docs.
Distill every finding into the appropriate S2 deliverable.

### During S3/S4 (future)

S2 is the primary source. Do **not** read S1 unless:

1. An S2 doc is ambiguous or contradictory on a specific point
2. A parity test reveals behavior not covered by S2
3. A human explicitly requests S1 verification

When any of these occur, read only the specific S1 atom needed
(~600 tokens each), resolve the gap, and **update S2** so the
same lookup is never needed again.

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

- Skip a stratum's exit gate (S2: phase 2.10, S3: parity green)
- **During S2:** make behavioral claims not traceable to S1 atoms
- **During S2:** produce deliverables that contradict S1 evidence
- **During S3/S4:** bulk-read S1 instead of reading S2
- **During S3/S4:** leave an S1 gap un-distilled into S2

## S3 Rules (activate when S3 is ACTIVE)

These rules are dormant while S2 is the active stratum.
They apply once S3/COOK becomes ACTIVE.

### S3 Context Discipline

**Read S2. Not S1.** All implementation decisions derive from S2
docs. The S2 deliverables are the contract:

- [architecture.md](docs/01-rr/s2/architecture.md) — crate map
- [scope.md](docs/01-rr/s2/scope.md) — MoSCoW tiers
- [invariants.md](docs/01-rr/s2/invariants.md) — behavioral rules
- [dependency-decisions.md](docs/01-rr/s2/dependency-decisions.md)
  — approved libraries
- [risks.md](docs/01-rr/s2/risks.md) — known hazards
- [ADRs](docs/01-rr/s2/adr/) — architectural decisions

If an S2 doc does not answer a question needed for implementation,
do not guess or read S1. Instead: flag the gap, add a TODO in the
S2 doc, and work with a human to fill it.

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
