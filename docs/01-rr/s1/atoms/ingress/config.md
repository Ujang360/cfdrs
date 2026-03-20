# Behavior Atom: ingress/config.go

## Source Anchor

- Go source: [cloudflare/cloudflared@2026.3.0/ingress/config.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/ingress/config.go)
- Package: ingress
- Module group: ingress

## Behavioral Responsibility

Ingress matching and origin dispatch behavior.

## Entry Points

- NewWarpRoutingConfig(raw *config.WarpRoutingConfig) WarpRoutingConfig (line 54)
- (*WarpRoutingConfig) RawConfig() config.WarpRoutingConfig (line 72)
- (*RemoteConfig) UnmarshalJSON(b []byte) error (line 98)
- ConvertToRawOriginConfig(c OriginRequestConfig) config.OriginRequestConfig (line 492)

## Internal Function Surface

- originRequestFromSingleRule(c *cli.Context) OriginRequestConfig (line 122)
- originRequestFromConfig(c config.OriginRequestConfig) OriginRequestConfig (line 215)
- (*OriginRequestConfig) setConnectTimeout(overrides config.OriginRequestConfig) (line 338)
- (*OriginRequestConfig) setTLSTimeout(overrides config.OriginRequestConfig) (line 344)
- (*OriginRequestConfig) setNoHappyEyeballs(overrides config.OriginRequestConfig) (line 350)
- (*OriginRequestConfig) setKeepAliveConnections(overrides config.OriginRequestConfig) (line 356)
- (*OriginRequestConfig) setKeepAliveTimeout(overrides config.OriginRequestConfig) (line 362)
- (*OriginRequestConfig) setTCPKeepAlive(overrides config.OriginRequestConfig) (line 368)
- (*OriginRequestConfig) setHTTPHostHeader(overrides config.OriginRequestConfig) (line 374)
- (*OriginRequestConfig) setOriginServerName(overrides config.OriginRequestConfig) (line 380)
- (*OriginRequestConfig) setMatchSNIToHost(overrides config.OriginRequestConfig) (line 386)
- (*OriginRequestConfig) setCAPool(overrides config.OriginRequestConfig) (line 392)
- (*OriginRequestConfig) setNoTLSVerify(overrides config.OriginRequestConfig) (line 398)
- (*OriginRequestConfig) setDisableChunkedEncoding(overrides config.OriginRequestConfig) (line 404)
- (*OriginRequestConfig) setBastionMode(overrides config.OriginRequestConfig) (line 410)
- (*OriginRequestConfig) setProxyPort(overrides config.OriginRequestConfig) (line 416)
- (*OriginRequestConfig) setProxyAddress(overrides config.OriginRequestConfig) (line 422)
- (*OriginRequestConfig) setProxyType(overrides config.OriginRequestConfig) (line 428)
- (*OriginRequestConfig) setIPRules(overrides config.OriginRequestConfig) (line 434)
- (*OriginRequestConfig) setHttp2Origin(overrides config.OriginRequestConfig) (line 447)
- (*OriginRequestConfig) setAccess(overrides config.OriginRequestConfig) (line 453)
- setConfig(defaults OriginRequestConfig, overrides config.OriginRequestConfig) OriginRequestConfig (line 467)
- convertToRawIPRules(ipRules []ipaccess.Rule) []config.IngressIPRule (line 546)
- defaultBoolToNil(b bool) *bool (line 563)
- emptyStringToNil(s string) *string (line 571)
- zeroUIntToNil(v uint) *uint (line 579)

## Input Contract

- CLI flags and command arguments
- func-param:b []byte
- func-param:b bool
- func-param:c *cli.Context
- func-param:c OriginRequestConfig
- func-param:c config.OriginRequestConfig
- func-param:defaults OriginRequestConfig
- func-param:ipRules []ipaccess.Rule
- func-param:overrides config.OriginRequestConfig
- func-param:raw *config.WarpRoutingConfig
- func-param:s string
- func-param:v uint
- serialized configuration payloads

## Output Contract

- return:*bool
- return:*string
- return:*uint
- return:OriginRequestConfig
- return:WarpRoutingConfig
- return:[]config.IngressIPRule
- return:config.OriginRequestConfig
- return:config.WarpRoutingConfig
- return:error

## Side Effects and State Transitions

- network I/O

## Branching and Failure Semantics

- Branch density: if=76, switch=0, select=0
- error-return paths

## Import and Dependency Surface

- encoding/json
- github.com/cloudflare/cloudflared/config
- github.com/cloudflare/cloudflared/ipaccess
- github.com/cloudflare/cloudflared/tlsconfig
- github.com/urfave/cli/v2
- time

## Go-Impl Flow (Intra-file)

```mermaid
flowchart TD
    F1["NewWarpRoutingConfig"]
    F2["*WarpRoutingConfig.RawConfig"]
    F3["*RemoteConfig.UnmarshalJSON"]
    F4["originRequestFromSingleRule"]
    F5["originRequestFromConfig"]
    F6["*OriginRequestConfig.setConnectTimeout"]
    F7["*OriginRequestConfig.setTLSTimeout"]
    F8["*OriginRequestConfig.setNoHappyEyeballs"]
    F9["*OriginRequestConfig.setKeepAliveConnections"]
    F10["*OriginRequestConfig.setKeepAliveTimeout"]
    F11["*OriginRequestConfig.setTCPKeepAlive"]
    F12["*OriginRequestConfig.setHTTPHostHeader"]
    F13["*OriginRequestConfig.setOriginServerName"]
    F14["*OriginRequestConfig.setMatchSNIToHost"]
    F15["*OriginRequestConfig.setCAPool"]
    F16["*OriginRequestConfig.setNoTLSVerify"]
    F17["*OriginRequestConfig.setDisableChunkedEncoding"]
    F18["*OriginRequestConfig.setBastionMode"]
    F19["*OriginRequestConfig.setProxyPort"]
    F20["*OriginRequestConfig.setProxyAddress"]
    F21["*OriginRequestConfig.setProxyType"]
    F22["*OriginRequestConfig.setIPRules"]
    F23["*OriginRequestConfig.setHttp2Origin"]
    F24["*OriginRequestConfig.setAccess"]
    F25["setConfig"]
    F26["ConvertToRawOriginConfig"]
    F27["convertToRawIPRules"]
    F28["defaultBoolToNil"]
    F29["emptyStringToNil"]
    F30["zeroUIntToNil"]
    F3 --> F5
    F3 --> F1
    F26 --> F28
    F26 --> F29
    F26 --> F30
    F26 --> F27
```

## Rust Porting Notes

- **Deeply nested option chains**: 76 conditionals for nil-guarding nested config fields → `Option<T>` with `.map()`, `.unwrap_or_default()`, and `?` operator chains.
- **yaml unmarshaling**: `yaml.Unmarshal` into nested structs → `serde_yaml::from_str()` with `#[derive(Deserialize)]`.
- **Default config builder**: `setDefaultValues()` mutating config → `Default` trait implementation or builder pattern with `.with_*()` methods.
- **Quirk — 76 if-branches**: Heavy null-checking; Rust’s `Option` eliminates most but requires careful `.is_some()` / `.as_ref()` patterns.

## Accuracy Notes

- Generated from Go AST parsing and source text pattern extraction.
- Source link is authoritative for disputed semantics; keep this atom synchronized with the linked file.
