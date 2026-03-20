# Behavior Atom: token/launch_browser_unix.go

## Source Anchor

- Go source: [cloudflare/cloudflared@2026.3.0/token/launch_browser_unix.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/token/launch_browser_unix.go)
- Package: token
- Module group: token

## Behavioral Responsibility

Configuration, identity, and credential handling behavior.

## Entry Points

- No exported/main/init entry point detected; behavior is internal support logic.

## Internal Function Surface

- getBrowserCmd(url string) *exec.Cmd (line 9)

## Input Contract

- func-param:url string

## Output Contract

- return:*exec.Cmd

## Side Effects and State Transitions

- subprocess execution

## Branching and Failure Semantics

- Branch density: if=0, switch=0, select=0
- No explicit failure pattern markers found in static scan.

## Import and Dependency Surface

- os/exec

## Go-Impl Flow (Intra-file)

```mermaid
flowchart TD
    F1["getBrowserCmd"]
    NOTE["No intra-file call edges detected; behavior may delegate externally"]
    F1 --> NOTE
```

## Rust Porting Notes

- **Build tag**: `//go:build linux || freebsd` → `#[cfg(any(target_os = "linux", target_os = "freebsd"))]`.
- **Browser launch**: `exec.Command("xdg-open", url)` → `std::process::Command::new("xdg-open").arg(url).spawn()`.

## Accuracy Notes

- Generated from Go AST parsing and source text pattern extraction.
- Source link is authoritative for disputed semantics; keep this atom synchronized with the linked file.
