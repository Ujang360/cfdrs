# Behavior Atom: diagnostic/system_collector_macos.go

## Source Anchor

- Go source: [cloudflare/cloudflared@2026.3.0/diagnostic/system_collector_macos.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/diagnostic/system_collector_macos.go)
- Package: diagnostic
- Module group: diagnostic

## Behavioral Responsibility

Management, diagnostics, and observability behavior.

## Entry Points

- NewSystemCollectorImpl(version string) *SystemCollectorImpl (line 17)
- (*SystemCollectorImpl) Collect(ctx context.Context) (*SystemInformation, error) (line 25)

## Internal Function Surface

- collectFileDescriptorInformation(ctx context.Context) (*FileDescriptorInformation, string, error) (line 101)
- collectMemoryInformation(ctx context.Context) (*MemoryInformation, string, error) (line 134)

## Input Contract

- func-param:ctx context.Context
- func-param:version string

## Output Contract

- return:*FileDescriptorInformation
- return:*MemoryInformation
- return:*SystemCollectorImpl
- return:*SystemInformation
- return:error
- return:string

## Side Effects and State Transitions

- subprocess execution

## Branching and Failure Semantics

- Branch density: if=8, switch=0, select=0
- error-return paths

## Import and Dependency Surface

- context
- fmt
- os/exec
- runtime
- strconv

## Go-Impl Flow (Intra-file)

```mermaid
flowchart TD
    F1["NewSystemCollectorImpl"]
    F2["*SystemCollectorImpl.Collect"]
    F3["collectFileDescriptorInformation"]
    F4["collectMemoryInformation"]
    F2 --> F4
    F2 --> F3
```

## Rust Porting Notes

- **sysctl parsing**: `sysctl -a` output parsing for system info → `sysctl` crate or `tokio::process::Command::new("sysctl")` with output parsing.
- **Build tag**: `//go:build darwin` → `#[cfg(target_os = "macos")]`.
- **Quirk — 8 if-branches**: Key-value parsing from sysctl output; reuse generic KV parser.

## Accuracy Notes

- Generated from Go AST parsing and source text pattern extraction.
- Source link is authoritative for disputed semantics; keep this atom synchronized with the linked file.
