# Behavior Atom: tunnelrpc/proto/quic_metadata_protocol.capnp.go

## Source Anchor

- Go source: [cloudflare/cloudflared@2026.3.0/tunnelrpc/proto/quic_metadata_protocol.capnp.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/tunnelrpc/proto/quic_metadata_protocol.capnp.go)
- Package: proto
- Module group: tunnelrpc

## Behavioral Responsibility

Core package behavior anchored to this source file.

## Entry Points

- NewConnectRequest(s *capnp.Segment) (ConnectRequest, error) (line 16)
- NewRootConnectRequest(s *capnp.Segment) (ConnectRequest, error) (line 21)
- ReadRootConnectRequest(msg *capnp.Message) (ConnectRequest, error) (line 26)
- (ConnectRequest) String() string (line 31)
- (ConnectRequest) Dest() (string, error) (line 36)
- (ConnectRequest) HasDest() bool (line 41)
- (ConnectRequest) DestBytes() ([]byte, error) (line 46)
- (ConnectRequest) SetDest(v string) error (line 51)
- (ConnectRequest) Type() ConnectionType (line 55)
- (ConnectRequest) SetType(v ConnectionType) (line 59)
- (ConnectRequest) Metadata() (Metadata_List, error) (line 63)
- (ConnectRequest) HasMetadata() bool (line 68)
- (ConnectRequest) SetMetadata(v Metadata_List) error (line 73)
- (ConnectRequest) NewMetadata(n int32) (Metadata_List, error) (line 79)
- NewConnectRequest_List(s *capnp.Segment, sz int32) (ConnectRequest_List, error) (line 92)
- (ConnectRequest_List) At(i int) ConnectRequest (line 97)
- (ConnectRequest_List) Set(i int, v ConnectRequest) error (line 99)
- (ConnectRequest_List) String() string (line 101)
- (ConnectRequest_Promise) Struct() (ConnectRequest, error) (line 109)
- (ConnectionType) String() string (line 127)
- ConnectionTypeFromString(c string) ConnectionType (line 143)
- NewConnectionType_List(s *capnp.Segment, sz int32) (ConnectionType_List, error) (line 159)
- (ConnectionType_List) At(i int) ConnectionType (line 164)
- (ConnectionType_List) Set(i int, v ConnectionType) (line 169)
- NewMetadata(s *capnp.Segment) (Metadata, error) (line 179)
- NewRootMetadata(s *capnp.Segment) (Metadata, error) (line 184)
- ReadRootMetadata(msg *capnp.Message) (Metadata, error) (line 189)
- (Metadata) String() string (line 194)
- (Metadata) Key() (string, error) (line 199)
- (Metadata) HasKey() bool (line 204)
- (Metadata) KeyBytes() ([]byte, error) (line 209)
- (Metadata) SetKey(v string) error (line 214)
- (Metadata) Val() (string, error) (line 218)
- (Metadata) HasVal() bool (line 223)
- (Metadata) ValBytes() ([]byte, error) (line 228)
- (Metadata) SetVal(v string) error (line 233)
- NewMetadata_List(s *capnp.Segment, sz int32) (Metadata_List, error) (line 241)
- (Metadata_List) At(i int) Metadata (line 246)
- (Metadata_List) Set(i int, v Metadata) error (line 248)
- (Metadata_List) String() string (line 250)
- (Metadata_Promise) Struct() (Metadata, error) (line 258)
- NewConnectResponse(s *capnp.Segment) (ConnectResponse, error) (line 268)
- NewRootConnectResponse(s *capnp.Segment) (ConnectResponse, error) (line 273)
- ReadRootConnectResponse(msg *capnp.Message) (ConnectResponse, error) (line 278)
- (ConnectResponse) String() string (line 283)
- (ConnectResponse) Error() (string, error) (line 288)
- (ConnectResponse) HasError() bool (line 293)
- (ConnectResponse) ErrorBytes() ([]byte, error) (line 298)
- (ConnectResponse) SetError(v string) error (line 303)
- (ConnectResponse) Metadata() (Metadata_List, error) (line 307)
- (ConnectResponse) HasMetadata() bool (line 312)
- (ConnectResponse) SetMetadata(v Metadata_List) error (line 317)
- (ConnectResponse) NewMetadata(n int32) (Metadata_List, error) (line 323)
- NewConnectResponse_List(s *capnp.Segment, sz int32) (ConnectResponse_List, error) (line 336)
- (ConnectResponse_List) At(i int) ConnectResponse (line 341)
- (ConnectResponse_List) Set(i int, v ConnectResponse) error (line 343)
- (ConnectResponse_List) String() string (line 347)
- (ConnectResponse_Promise) Struct() (ConnectResponse, error) (line 355)
- init() (line 389)

## Internal Function Surface

- None detected.

## Input Contract

- func-param:c string
- func-param:i int
- func-param:msg *capnp.Message
- func-param:n int32
- func-param:s *capnp.Segment
- func-param:sz int32
- func-param:v ConnectRequest
- func-param:v ConnectResponse
- func-param:v ConnectionType
- func-param:v Metadata
- func-param:v Metadata_List
- func-param:v string

## Output Contract

- return:ConnectRequest
- return:ConnectRequest_List
- return:ConnectResponse
- return:ConnectResponse_List
- return:ConnectionType
- return:ConnectionType_List
- return:Metadata
- return:Metadata_List
- return:[]byte
- return:bool
- return:error
- return:string

## Side Effects and State Transitions

- No high-signal side effect pattern detected in static scan.

## Branching and Failure Semantics

- Branch density: if=2, switch=2, select=0
- error-return paths
- fallback/default branches

## Import and Dependency Surface

- zombiezen.com/go/capnproto2
- zombiezen.com/go/capnproto2/encoding/text
- zombiezen.com/go/capnproto2/schemas

## Go-Impl Flow (Intra-file)

```mermaid
flowchart TD
    F1["NewConnectRequest"]
    F2["NewRootConnectRequest"]
    F3["ReadRootConnectRequest"]
    F4["ConnectRequest.String"]
    F5["ConnectRequest.Dest"]
    F6["ConnectRequest.HasDest"]
    F7["ConnectRequest.DestBytes"]
    F8["ConnectRequest.SetDest"]
    F9["ConnectRequest.Type"]
    F10["ConnectRequest.SetType"]
    F11["ConnectRequest.Metadata"]
    F12["ConnectRequest.HasMetadata"]
    F13["ConnectRequest.SetMetadata"]
    F14["ConnectRequest.NewMetadata"]
    F15["NewConnectRequest_List"]
    F16["ConnectRequest_List.At"]
    F17["ConnectRequest_List.Set"]
    F18["ConnectRequest_List.String"]
    F19["ConnectRequest_Promise.Struct"]
    F20["ConnectionType.String"]
    F21["ConnectionTypeFromString"]
    F22["NewConnectionType_List"]
    F23["ConnectionType_List.At"]
    F24["ConnectionType_List.Set"]
    F25["NewMetadata"]
    F26["NewRootMetadata"]
    F27["ReadRootMetadata"]
    F28["Metadata.String"]
    F29["Metadata.Key"]
    F30["Metadata.HasKey"]
    F31["Metadata.KeyBytes"]
    F32["Metadata.SetKey"]
    F33["Metadata.Val"]
    F34["Metadata.HasVal"]
    F35["Metadata.ValBytes"]
    F36["Metadata.SetVal"]
    F37["NewMetadata_List"]
    F38["Metadata_List.At"]
    F39["Metadata_List.Set"]
    F40["Metadata_List.String"]
    F41["Metadata_Promise.Struct"]
    F42["NewConnectResponse"]
    F43["NewRootConnectResponse"]
    F44["ReadRootConnectResponse"]
    F45["ConnectResponse.String"]
    F46["ConnectResponse.Error"]
    F47["ConnectResponse.HasError"]
    F48["ConnectResponse.ErrorBytes"]
    F49["ConnectResponse.SetError"]
    F50["ConnectResponse.Metadata"]
    F51["ConnectResponse.HasMetadata"]
    F52["ConnectResponse.SetMetadata"]
    F53["ConnectResponse.NewMetadata"]
    F54["NewConnectResponse_List"]
    F55["ConnectResponse_List.At"]
    F56["ConnectResponse_List.Set"]
    F57["ConnectResponse_List.String"]
    F58["ConnectResponse_Promise.Struct"]
    F59["init"]
    F14 --> F37
    F53 --> F37
```

## Rust Porting Notes

- **Generated code**: Entire file is generated from `quic_metadata_protocol.capnp` schema → regenerate via `capnpc-rust` (the `capnp` crate's code generator); do not hand-port.
- **Segment-based memory model**: Cap'n Proto's arena-allocated segments → `capnp::message::Builder` / `capnp::message::Reader` in Rust; access patterns differ from Go's pointer-based API.
- **Has* optional checks**: `HasField()` methods for optional Cap'n Proto fields → `reader.has_field()` in `capnp-rust`; maps naturally.
- **init() schema registration**: Go `init()` registers schemas → not needed in Rust; `capnp-rust` uses compile-time schema embedding.
- **Quirk — do not hand-edit**: This is generated code; the Rust port must regenerate from the `.capnp` schema file, not translate the Go output.

## Accuracy Notes

- Generated from Go AST parsing and source text pattern extraction.
- Source link is authoritative for disputed semantics; keep this atom synchronized with the linked file.
