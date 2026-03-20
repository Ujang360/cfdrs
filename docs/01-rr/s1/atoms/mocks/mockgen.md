# Behavior Atom: mocks/mockgen.go

## Source Anchor

- Go source: [cloudflare/cloudflared@2026.3.0/mocks/mockgen.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/mocks/mockgen.go)
- Package: mocks
- Module group: mocks

## Behavioral Responsibility

Core package behavior anchored to this source file.

## Entry Points

- No exported/main/init entry point detected; behavior is internal support logic.

## Internal Function Surface

- None detected.

## Input Contract

- Inputs are indirect through callers; no direct input pattern detected statically.

## Output Contract

- Output is primarily side-effect based; no explicit return/output pattern detected statically.

## Side Effects and State Transitions

- No high-signal side effect pattern detected in static scan.

## Branching and Failure Semantics

- Branch density: if=0, switch=0, select=0
- No explicit failure pattern markers found in static scan.

## Import and Dependency Surface

- No imports.

## Go-Impl Flow (Intra-file)

```mermaid
flowchart TD
    START["No functions parsed"]
```

## Rust Porting Notes

- **go:generate mockgen**: Drop entirely → use `mockall::automock` attribute on trait definitions or manual mock structs.

## Accuracy Notes

- Generated from Go AST parsing and source text pattern extraction.
- Source link is authoritative for disputed semantics; keep this atom synchronized with the linked file.
