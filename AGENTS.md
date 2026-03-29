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
2. **Never read `dependency-shopping-cart.md` in full.** It is a
   research artifact. Use
   [dependency-decisions.md](docs/01-rr/s2/dependency-decisions.md)
   as the authoritative summary (~6K tokens).
3. **Cap S2 reads to ≤3 docs per session.** If you need more than
   architecture + scope + one ADR, you are likely violating
   "one deliverable, one session."
4. **Prefer search over read.** When verifying a claim, grep for the
   keyword instead of reading the entire document.
5. **Budget check before large reads.** Before opening any file
   >10K tokens, confirm the read is necessary for the current
   deliverable.

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
