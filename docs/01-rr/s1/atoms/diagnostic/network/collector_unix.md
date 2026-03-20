# Behavior Atom: diagnostic/network/collector_unix.go

## Source Anchor

- Go source: [cloudflare/cloudflared@2026.3.0/diagnostic/network/collector_unix.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/diagnostic/network/collector_unix.go)
- Package: diagnostic
- Module group: diagnostic

## Behavioral Responsibility

Management, diagnostics, and observability behavior.

## Entry Points

- (*NetworkCollectorImpl) Collect(ctx context.Context, options TraceOptions) ([]*Hop, string, error) (line 16)
- DecodeLine(text string) (*Hop, error) (line 40)

## Internal Function Surface

- None detected.

## Input Contract

- func-param:ctx context.Context
- func-param:options TraceOptions
- func-param:text string

## Output Contract

- return:*Hop
- return:[]*Hop
- return:error
- return:string

## Side Effects and State Transitions

- subprocess execution

## Branching and Failure Semantics

- Branch density: if=5, switch=1, select=0
- error-return paths
- fallback/default branches

## Import and Dependency Surface

- context
- fmt
- os/exec
- strconv
- strings
- time

## Go-Impl Flow (Intra-file)

```mermaid
flowchart TD
    F1["*NetworkCollectorImpl.Collect"]
    F2["DecodeLine"]
    NOTE["No intra-file call edges detected; behavior may delegate externally"]
    F1 --> NOTE
```

## Rust Porting Notes

- **Traceroute parsing**: Parses `traceroute` command output with regex → `regex::Regex` captures for hop address/RTT extraction.
- **Build tag**: `//go:build unix` → `#[cfg(unix)]`.
- **strconv parsing**: String-to-number conversion errors → `str::parse::<f64>()` returning `Result`.
- **Quirk — 5 if-branches + 1 switch**: Line format detection; use `match` on parsed line structure.

## Accuracy Notes

- Generated from Go AST parsing and source text pattern extraction.
- Source link is authoritative for disputed semantics; keep this atom synchronized with the linked file.
