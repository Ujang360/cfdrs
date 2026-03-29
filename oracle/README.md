# Oracle Binary — cloudflared 2026.3.0

| | |
| --- | --- |
| Source | [cloudflare/cloudflared](https://github.com/cloudflare/cloudflared) @ tag `2026.3.0` |
| Platform | linux/amd64 |
| Binary | `cloudflared-amd64-linux` |
| SHA-256 | `b68e1c2734f87aee03e4937afa56e0102744c7faecd1cd75e3c3107a710f08e6` |
| Size | 26 753 680 bytes (~25.5 MiB) |
| Type | ELF 64-bit LSB executable, x86-64, dynamically linked |
| Build ID | `d99f298f82cbfd9b758363c717d8778a219309e7` |

## Purpose

The oracle binary is the behavioral ground truth for the
[cfdrs](../README.md) Rust rewrite. Parity tests (S3/S4) invoke this
binary to generate reference outputs for comparison against the Rust
implementation. It is never modified or rebuilt during testing.

## Acquisition

1. Clone [cloudflare/cloudflared](https://github.com/cloudflare/cloudflared)
   at tag `2026.3.0`
2. Build:

   ```sh
   GOOS=linux GOARCH=amd64 go build -o cloudflared-amd64-linux ./cmd/cloudflared
   ```

3. Verify hash:

   ```sh
   sha256sum cloudflared-amd64-linux
   # Expected: b68e1c2734f87aee03e4937afa56e0102744c7faecd1cd75e3c3107a710f08e6
   ```

4. Place in `oracle/cloudflared-amd64-linux`

## Git Policy

The binary is **gitignored** (too large for version control). The hash
above pins the exact build. If the binary is ever rebuilt, update this
README with the new hash.

The `oracle/captures/` directory (when created) is **git-tracked** — it
holds small text and binary fixtures produced from oracle runs.

## Usage in Parity Tests

Parity tests interact with the oracle binary through a subprocess model:

```text
Test code (Rust) → Oracle runner (subprocess) → Go binary (subprocess)
                ← Captured output             ← stdout/stderr/exit code
```

See [parity harness design](../docs/01-rr/s2/parity-design.md) for the
full oracle interaction model, including environment isolation, timeout
handling, and output normalization.
