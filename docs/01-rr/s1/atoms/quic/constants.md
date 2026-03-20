# Behavior Atom: quic/constants.go

## Source Anchor

- Go source: [cloudflare/cloudflared@2026.3.0/quic/constants.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/quic/constants.go)
- Package: quic
- Module group: quic

## Behavioral Responsibility

Transport/protocol behavior for edge-origin data and control flows.

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

- time

## Go-Impl Flow (Intra-file)

```mermaid
flowchart TD
    START["No functions parsed"]
```

## Rust Porting Notes

- **Constants file**: Package-level constants for QUIC transport parameters → Rust `const` items in a `quic::constants` module.
- **ALPN tokens**: String constants for TLS ALPN negotiation → `&[u8]` constants (ALPN is byte-oriented in `rustls`).
- **Quirk — no logic**: Pure constant definitions; the Rust port is a direct `const` transcription.

## Accuracy Notes

- Generated from Go AST parsing and source text pattern extraction.
- Source link is authoritative for disputed semantics; keep this atom synchronized with the linked file.
