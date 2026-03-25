# ADR-011 - EVL Architecture

| | |
| --- | --- |
| Baseline | cloudflare/cloudflared @ tag `2026.3.0` |
| Phase | 01-RR / S2 - SUBSTRATE |
| Status | Open — decided in S2.6 |
| Origin | [dependency-decisions](../dependency-decisions.md) § 1.1 and Layer 1 |
| Risk linkage | — |

## Question

What is the precise EVL architecture — thread count, pinning
policy, assignment strategy, and inter-EVL boundary rules?

## Decision

Deferred to S2.6 where crate boundaries are drawn from Jaccard
clusters. High-level constraints already locked in dependency-decisions: single
multi-thread tokio runtime, all threads pinned via `sched_setaffinity`, Tunnel
EVL (4 threads) + Proxy EVL (remaining cores), no connection migration, atomic
counter load balancing at assignment time. S2.6 will formalize these into typed
boundaries and crate-level ownership rules.

## Consequences

- This ADR must be resolved and closed before S2.6 exits. S3.0
  foundation work depends on the EVL boundary types being locked.
