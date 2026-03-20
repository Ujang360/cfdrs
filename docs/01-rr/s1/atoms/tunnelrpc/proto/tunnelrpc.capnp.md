# Behavior Atom: tunnelrpc/proto/tunnelrpc.capnp.go

## Source Anchor

- Go source: [cloudflare/cloudflared@2026.3.0/tunnelrpc/proto/tunnelrpc.capnp.go](https://github.com/cloudflare/cloudflared/blob/2026.3.0/tunnelrpc/proto/tunnelrpc.capnp.go)
- Package: proto
- Module group: tunnelrpc

## Behavioral Responsibility

Core package behavior anchored to this source file.

## Entry Points

- NewAuthentication(s *capnp.Segment) (Authentication, error) (line 20)
- NewRootAuthentication(s *capnp.Segment) (Authentication, error) (line 25)
- ReadRootAuthentication(msg *capnp.Message) (Authentication, error) (line 30)
- (Authentication) String() string (line 35)
- (Authentication) Key() (string, error) (line 40)
- (Authentication) HasKey() bool (line 45)
- (Authentication) KeyBytes() ([]byte, error) (line 50)
- (Authentication) SetKey(v string) error (line 55)
- (Authentication) Email() (string, error) (line 59)
- (Authentication) HasEmail() bool (line 64)
- (Authentication) EmailBytes() ([]byte, error) (line 69)
- (Authentication) SetEmail(v string) error (line 74)
- (Authentication) OriginCAKey() (string, error) (line 78)
- (Authentication) HasOriginCAKey() bool (line 83)
- (Authentication) OriginCAKeyBytes() ([]byte, error) (line 88)
- (Authentication) SetOriginCAKey(v string) error (line 93)
- NewAuthentication_List(s *capnp.Segment, sz int32) (Authentication_List, error) (line 101)
- (Authentication_List) At(i int) Authentication (line 106)
- (Authentication_List) Set(i int, v Authentication) error (line 108)
- (Authentication_List) String() string (line 110)
- (Authentication_Promise) Struct() (Authentication, error) (line 118)
- NewTunnelRegistration(s *capnp.Segment) (TunnelRegistration, error) (line 128)
- NewRootTunnelRegistration(s *capnp.Segment) (TunnelRegistration, error) (line 133)
- ReadRootTunnelRegistration(msg *capnp.Message) (TunnelRegistration, error) (line 138)
- (TunnelRegistration) String() string (line 143)
- (TunnelRegistration) Err() (string, error) (line 148)
- (TunnelRegistration) HasErr() bool (line 153)
- (TunnelRegistration) ErrBytes() ([]byte, error) (line 158)
- (TunnelRegistration) SetErr(v string) error (line 163)
- (TunnelRegistration) Url() (string, error) (line 167)
- (TunnelRegistration) HasUrl() bool (line 172)
- (TunnelRegistration) UrlBytes() ([]byte, error) (line 177)
- (TunnelRegistration) SetUrl(v string) error (line 182)
- (TunnelRegistration) LogLines() (capnp.TextList, error) (line 186)
- (TunnelRegistration) HasLogLines() bool (line 191)
- (TunnelRegistration) SetLogLines(v capnp.TextList) error (line 196)
- (TunnelRegistration) NewLogLines(n int32) (capnp.TextList, error) (line 202)
- (TunnelRegistration) PermanentFailure() bool (line 211)
- (TunnelRegistration) SetPermanentFailure(v bool) (line 215)
- (TunnelRegistration) TunnelID() (string, error) (line 219)
- (TunnelRegistration) HasTunnelID() bool (line 224)
- (TunnelRegistration) TunnelIDBytes() ([]byte, error) (line 229)
- (TunnelRegistration) SetTunnelID(v string) error (line 234)
- (TunnelRegistration) RetryAfterSeconds() uint16 (line 238)
- (TunnelRegistration) SetRetryAfterSeconds(v uint16) (line 242)
- (TunnelRegistration) EventDigest() ([]byte, error) (line 246)
- (TunnelRegistration) HasEventDigest() bool (line 251)
- (TunnelRegistration) SetEventDigest(v []byte) error (line 256)
- (TunnelRegistration) ConnDigest() ([]byte, error) (line 260)
- (TunnelRegistration) HasConnDigest() bool (line 265)
- (TunnelRegistration) SetConnDigest(v []byte) error (line 270)
- NewTunnelRegistration_List(s *capnp.Segment, sz int32) (TunnelRegistration_List, error) (line 278)
- (TunnelRegistration_List) At(i int) TunnelRegistration (line 283)
- (TunnelRegistration_List) Set(i int, v TunnelRegistration) error (line 287)
- (TunnelRegistration_List) String() string (line 291)
- (TunnelRegistration_Promise) Struct() (TunnelRegistration, error) (line 299)
- NewRegistrationOptions(s *capnp.Segment) (RegistrationOptions, error) (line 309)
- NewRootRegistrationOptions(s *capnp.Segment) (RegistrationOptions, error) (line 314)
- ReadRootRegistrationOptions(msg *capnp.Message) (RegistrationOptions, error) (line 319)
- (RegistrationOptions) String() string (line 324)
- (RegistrationOptions) ClientId() (string, error) (line 329)
- (RegistrationOptions) HasClientId() bool (line 334)
- (RegistrationOptions) ClientIdBytes() ([]byte, error) (line 339)
- (RegistrationOptions) SetClientId(v string) error (line 344)
- (RegistrationOptions) Version() (string, error) (line 348)
- (RegistrationOptions) HasVersion() bool (line 353)
- (RegistrationOptions) VersionBytes() ([]byte, error) (line 358)
- (RegistrationOptions) SetVersion(v string) error (line 363)
- (RegistrationOptions) Os() (string, error) (line 367)
- (RegistrationOptions) HasOs() bool (line 372)
- (RegistrationOptions) OsBytes() ([]byte, error) (line 377)
- (RegistrationOptions) SetOs(v string) error (line 382)
- (RegistrationOptions) ExistingTunnelPolicy() ExistingTunnelPolicy (line 386)
- (RegistrationOptions) SetExistingTunnelPolicy(v ExistingTunnelPolicy) (line 390)
- (RegistrationOptions) PoolName() (string, error) (line 394)
- (RegistrationOptions) HasPoolName() bool (line 399)
- (RegistrationOptions) PoolNameBytes() ([]byte, error) (line 404)
- (RegistrationOptions) SetPoolName(v string) error (line 409)
- (RegistrationOptions) Tags() (Tag_List, error) (line 413)
- (RegistrationOptions) HasTags() bool (line 418)
- (RegistrationOptions) SetTags(v Tag_List) error (line 423)
- (RegistrationOptions) NewTags(n int32) (Tag_List, error) (line 429)
- (RegistrationOptions) ConnectionId() uint8 (line 438)
- (RegistrationOptions) SetConnectionId(v uint8) (line 442)
- (RegistrationOptions) OriginLocalIp() (string, error) (line 446)
- (RegistrationOptions) HasOriginLocalIp() bool (line 451)
- (RegistrationOptions) OriginLocalIpBytes() ([]byte, error) (line 456)
- (RegistrationOptions) SetOriginLocalIp(v string) error (line 461)
- (RegistrationOptions) IsAutoupdated() bool (line 465)
- (RegistrationOptions) SetIsAutoupdated(v bool) (line 469)
- (RegistrationOptions) RunFromTerminal() bool (line 473)
- (RegistrationOptions) SetRunFromTerminal(v bool) (line 477)
- (RegistrationOptions) CompressionQuality() uint64 (line 481)
- (RegistrationOptions) SetCompressionQuality(v uint64) (line 485)
- (RegistrationOptions) Uuid() (string, error) (line 489)
- (RegistrationOptions) HasUuid() bool (line 494)
- (RegistrationOptions) UuidBytes() ([]byte, error) (line 499)
- (RegistrationOptions) SetUuid(v string) error (line 504)
- (RegistrationOptions) NumPreviousAttempts() uint8 (line 508)
- (RegistrationOptions) SetNumPreviousAttempts(v uint8) (line 512)
- (RegistrationOptions) Features() (capnp.TextList, error) (line 516)
- (RegistrationOptions) HasFeatures() bool (line 521)
- (RegistrationOptions) SetFeatures(v capnp.TextList) error (line 526)
- (RegistrationOptions) NewFeatures(n int32) (capnp.TextList, error) (line 532)
- NewRegistrationOptions_List(s *capnp.Segment, sz int32) (RegistrationOptions_List, error) (line 545)
- (RegistrationOptions_List) At(i int) RegistrationOptions (line 550)
- (RegistrationOptions_List) Set(i int, v RegistrationOptions) error (line 554)
- (RegistrationOptions_List) String() string (line 558)
- (RegistrationOptions_Promise) Struct() (RegistrationOptions, error) (line 566)
- (ExistingTunnelPolicy) String() string (line 584)
- ExistingTunnelPolicyFromString(c string) ExistingTunnelPolicy (line 600)
- NewExistingTunnelPolicy_List(s *capnp.Segment, sz int32) (ExistingTunnelPolicy_List, error) (line 616)
- (ExistingTunnelPolicy_List) At(i int) ExistingTunnelPolicy (line 621)
- (ExistingTunnelPolicy_List) Set(i int, v ExistingTunnelPolicy) (line 626)
- NewServerInfo(s *capnp.Segment) (ServerInfo, error) (line 636)
- NewRootServerInfo(s *capnp.Segment) (ServerInfo, error) (line 641)
- ReadRootServerInfo(msg *capnp.Message) (ServerInfo, error) (line 646)
- (ServerInfo) String() string (line 651)
- (ServerInfo) LocationName() (string, error) (line 656)
- (ServerInfo) HasLocationName() bool (line 661)
- (ServerInfo) LocationNameBytes() ([]byte, error) (line 666)
- (ServerInfo) SetLocationName(v string) error (line 671)
- NewServerInfo_List(s *capnp.Segment, sz int32) (ServerInfo_List, error) (line 679)
- (ServerInfo_List) At(i int) ServerInfo (line 684)
- (ServerInfo_List) Set(i int, v ServerInfo) error (line 686)
- (ServerInfo_List) String() string (line 688)
- (ServerInfo_Promise) Struct() (ServerInfo, error) (line 696)
- NewAuthenticateResponse(s *capnp.Segment) (AuthenticateResponse, error) (line 706)
- NewRootAuthenticateResponse(s *capnp.Segment) (AuthenticateResponse, error) (line 711)
- ReadRootAuthenticateResponse(msg *capnp.Message) (AuthenticateResponse, error) (line 716)
- (AuthenticateResponse) String() string (line 721)
- (AuthenticateResponse) PermanentErr() (string, error) (line 726)
- (AuthenticateResponse) HasPermanentErr() bool (line 731)
- (AuthenticateResponse) PermanentErrBytes() ([]byte, error) (line 736)
- (AuthenticateResponse) SetPermanentErr(v string) error (line 741)
- (AuthenticateResponse) RetryableErr() (string, error) (line 745)
- (AuthenticateResponse) HasRetryableErr() bool (line 750)
- (AuthenticateResponse) RetryableErrBytes() ([]byte, error) (line 755)
- (AuthenticateResponse) SetRetryableErr(v string) error (line 760)
- (AuthenticateResponse) Jwt() ([]byte, error) (line 764)
- (AuthenticateResponse) HasJwt() bool (line 769)
- (AuthenticateResponse) SetJwt(v []byte) error (line 774)
- (AuthenticateResponse) HoursUntilRefresh() uint8 (line 778)
- (AuthenticateResponse) SetHoursUntilRefresh(v uint8) (line 782)
- NewAuthenticateResponse_List(s *capnp.Segment, sz int32) (AuthenticateResponse_List, error) (line 790)
- (AuthenticateResponse_List) At(i int) AuthenticateResponse (line 795)
- (AuthenticateResponse_List) Set(i int, v AuthenticateResponse) error (line 799)
- (AuthenticateResponse_List) String() string (line 803)
- (AuthenticateResponse_Promise) Struct() (AuthenticateResponse, error) (line 811)
- (TunnelServer) RegisterTunnel(ctx context.Context, params func(TunnelServer_registerTunnel_Params) error, opts ...capnp.CallOption) TunnelServer_registerTunnel_Results_Promise (line 821)
- (TunnelServer) GetServerInfo(ctx context.Context, params func(TunnelServer_getServerInfo_Params) error, opts ...capnp.CallOption) TunnelServer_getServerInfo_Results_Promise (line 841)
- (TunnelServer) UnregisterTunnel(ctx context.Context, params func(TunnelServer_unregisterTunnel_Params) error, opts ...capnp.CallOption) TunnelServer_unregisterTunnel_Results_Promise (line 861)
- (TunnelServer) ObsoleteDeclarativeTunnelConnect(ctx context.Context, params func(TunnelServer_obsoleteDeclarativeTunnelConnect_Params) error, opts ...capnp.CallOption) TunnelServer_obsoleteDeclarativeTunnelConnect_Results_Promise (line 881)
- (TunnelServer) Authenticate(ctx context.Context, params func(TunnelServer_authenticate_Params) error, opts ...capnp.CallOption) TunnelServer_authenticate_Results_Promise (line 903)
- (TunnelServer) ReconnectTunnel(ctx context.Context, params func(TunnelServer_reconnectTunnel_Params) error, opts ...capnp.CallOption) TunnelServer_reconnectTunnel_Results_Promise (line 923)
- (TunnelServer) RegisterConnection(ctx context.Context, params func(RegistrationServer_registerConnection_Params) error, opts ...capnp.CallOption) RegistrationServer_registerConnection_Results_Promise (line 943)
- (TunnelServer) UnregisterConnection(ctx context.Context, params func(RegistrationServer_unregisterConnection_Params) error, opts ...capnp.CallOption) RegistrationServer_unregisterConnection_Results_Promise (line 963)
- (TunnelServer) UpdateLocalConfiguration(ctx context.Context, params func(RegistrationServer_updateLocalConfiguration_Params) error, opts ...capnp.CallOption) RegistrationServer_updateLocalConfiguration_Results_Promise (line 983)
- TunnelServer_ServerToClient(s TunnelServer_Server) TunnelServer (line 1026)
- TunnelServer_Methods(methods []server.Method, s TunnelServer_Server) []server.Method (line 1031)
- NewTunnelServer_registerTunnel_Params(s *capnp.Segment) (TunnelServer_registerTunnel_Params, error) (line 1218)
- NewRootTunnelServer_registerTunnel_Params(s *capnp.Segment) (TunnelServer_registerTunnel_Params, error) (line 1223)
- ReadRootTunnelServer_registerTunnel_Params(msg *capnp.Message) (TunnelServer_registerTunnel_Params, error) (line 1228)
- (TunnelServer_registerTunnel_Params) String() string (line 1233)
- (TunnelServer_registerTunnel_Params) OriginCert() ([]byte, error) (line 1238)
- (TunnelServer_registerTunnel_Params) HasOriginCert() bool (line 1243)
- (TunnelServer_registerTunnel_Params) SetOriginCert(v []byte) error (line 1248)
- (TunnelServer_registerTunnel_Params) Hostname() (string, error) (line 1252)
- (TunnelServer_registerTunnel_Params) HasHostname() bool (line 1257)
- (TunnelServer_registerTunnel_Params) HostnameBytes() ([]byte, error) (line 1262)
- (TunnelServer_registerTunnel_Params) SetHostname(v string) error (line 1267)
- (TunnelServer_registerTunnel_Params) Options() (RegistrationOptions, error) (line 1271)
- (TunnelServer_registerTunnel_Params) HasOptions() bool (line 1276)
- (TunnelServer_registerTunnel_Params) SetOptions(v RegistrationOptions) error (line 1281)
- (TunnelServer_registerTunnel_Params) NewOptions() (RegistrationOptions, error) (line 1287)
- NewTunnelServer_registerTunnel_Params_List(s *capnp.Segment, sz int32) (TunnelServer_registerTunnel_Params_List, error) (line 1300)
- (TunnelServer_registerTunnel_Params_List) At(i int) TunnelServer_registerTunnel_Params (line 1305)
- (TunnelServer_registerTunnel_Params_List) Set(i int, v TunnelServer_registerTunnel_Params) error (line 1309)
- (TunnelServer_registerTunnel_Params_List) String() string (line 1313)
- (TunnelServer_registerTunnel_Params_Promise) Struct() (TunnelServer_registerTunnel_Params, error) (line 1321)
- (TunnelServer_registerTunnel_Params_Promise) Options() RegistrationOptions_Promise (line 1326)
- NewTunnelServer_registerTunnel_Results(s *capnp.Segment) (TunnelServer_registerTunnel_Results, error) (line 1335)
- NewRootTunnelServer_registerTunnel_Results(s *capnp.Segment) (TunnelServer_registerTunnel_Results, error) (line 1340)
- ReadRootTunnelServer_registerTunnel_Results(msg *capnp.Message) (TunnelServer_registerTunnel_Results, error) (line 1345)
- (TunnelServer_registerTunnel_Results) String() string (line 1350)
- (TunnelServer_registerTunnel_Results) Result() (TunnelRegistration, error) (line 1355)
- (TunnelServer_registerTunnel_Results) HasResult() bool (line 1360)
- (TunnelServer_registerTunnel_Results) SetResult(v TunnelRegistration) error (line 1365)
- (TunnelServer_registerTunnel_Results) NewResult() (TunnelRegistration, error) (line 1371)
- NewTunnelServer_registerTunnel_Results_List(s *capnp.Segment, sz int32) (TunnelServer_registerTunnel_Results_List, error) (line 1384)
- (TunnelServer_registerTunnel_Results_List) At(i int) TunnelServer_registerTunnel_Results (line 1389)
- (TunnelServer_registerTunnel_Results_List) Set(i int, v TunnelServer_registerTunnel_Results) error (line 1393)
- (TunnelServer_registerTunnel_Results_List) String() string (line 1397)
- (TunnelServer_registerTunnel_Results_Promise) Struct() (TunnelServer_registerTunnel_Results, error) (line 1405)
- (TunnelServer_registerTunnel_Results_Promise) Result() TunnelRegistration_Promise (line 1410)
- NewTunnelServer_getServerInfo_Params(s *capnp.Segment) (TunnelServer_getServerInfo_Params, error) (line 1419)
- NewRootTunnelServer_getServerInfo_Params(s *capnp.Segment) (TunnelServer_getServerInfo_Params, error) (line 1424)
- ReadRootTunnelServer_getServerInfo_Params(msg *capnp.Message) (TunnelServer_getServerInfo_Params, error) (line 1429)
- (TunnelServer_getServerInfo_Params) String() string (line 1434)
- NewTunnelServer_getServerInfo_Params_List(s *capnp.Segment, sz int32) (TunnelServer_getServerInfo_Params_List, error) (line 1443)
- (TunnelServer_getServerInfo_Params_List) At(i int) TunnelServer_getServerInfo_Params (line 1448)
- (TunnelServer_getServerInfo_Params_List) Set(i int, v TunnelServer_getServerInfo_Params) error (line 1452)
- (TunnelServer_getServerInfo_Params_List) String() string (line 1456)
- (TunnelServer_getServerInfo_Params_Promise) Struct() (TunnelServer_getServerInfo_Params, error) (line 1464)
- NewTunnelServer_getServerInfo_Results(s *capnp.Segment) (TunnelServer_getServerInfo_Results, error) (line 1474)
- NewRootTunnelServer_getServerInfo_Results(s *capnp.Segment) (TunnelServer_getServerInfo_Results, error) (line 1479)
- ReadRootTunnelServer_getServerInfo_Results(msg *capnp.Message) (TunnelServer_getServerInfo_Results, error) (line 1484)
- (TunnelServer_getServerInfo_Results) String() string (line 1489)
- (TunnelServer_getServerInfo_Results) Result() (ServerInfo, error) (line 1494)
- (TunnelServer_getServerInfo_Results) HasResult() bool (line 1499)
- (TunnelServer_getServerInfo_Results) SetResult(v ServerInfo) error (line 1504)
- (TunnelServer_getServerInfo_Results) NewResult() (ServerInfo, error) (line 1510)
- NewTunnelServer_getServerInfo_Results_List(s *capnp.Segment, sz int32) (TunnelServer_getServerInfo_Results_List, error) (line 1523)
- (TunnelServer_getServerInfo_Results_List) At(i int) TunnelServer_getServerInfo_Results (line 1528)
- (TunnelServer_getServerInfo_Results_List) Set(i int, v TunnelServer_getServerInfo_Results) error (line 1532)
- (TunnelServer_getServerInfo_Results_List) String() string (line 1536)
- (TunnelServer_getServerInfo_Results_Promise) Struct() (TunnelServer_getServerInfo_Results, error) (line 1544)
- (TunnelServer_getServerInfo_Results_Promise) Result() ServerInfo_Promise (line 1549)
- NewTunnelServer_unregisterTunnel_Params(s *capnp.Segment) (TunnelServer_unregisterTunnel_Params, error) (line 1558)
- NewRootTunnelServer_unregisterTunnel_Params(s *capnp.Segment) (TunnelServer_unregisterTunnel_Params, error) (line 1563)
- ReadRootTunnelServer_unregisterTunnel_Params(msg *capnp.Message) (TunnelServer_unregisterTunnel_Params, error) (line 1568)
- (TunnelServer_unregisterTunnel_Params) String() string (line 1573)
- (TunnelServer_unregisterTunnel_Params) GracePeriodNanoSec() int64 (line 1578)
- (TunnelServer_unregisterTunnel_Params) SetGracePeriodNanoSec(v int64) (line 1582)
- NewTunnelServer_unregisterTunnel_Params_List(s *capnp.Segment, sz int32) (TunnelServer_unregisterTunnel_Params_List, error) (line 1590)
- (TunnelServer_unregisterTunnel_Params_List) At(i int) TunnelServer_unregisterTunnel_Params (line 1595)
- (TunnelServer_unregisterTunnel_Params_List) Set(i int, v TunnelServer_unregisterTunnel_Params) error (line 1599)
- (TunnelServer_unregisterTunnel_Params_List) String() string (line 1603)
- (TunnelServer_unregisterTunnel_Params_Promise) Struct() (TunnelServer_unregisterTunnel_Params, error) (line 1611)
- NewTunnelServer_unregisterTunnel_Results(s *capnp.Segment) (TunnelServer_unregisterTunnel_Results, error) (line 1621)
- NewRootTunnelServer_unregisterTunnel_Results(s *capnp.Segment) (TunnelServer_unregisterTunnel_Results, error) (line 1626)
- ReadRootTunnelServer_unregisterTunnel_Results(msg *capnp.Message) (TunnelServer_unregisterTunnel_Results, error) (line 1631)
- (TunnelServer_unregisterTunnel_Results) String() string (line 1636)
- NewTunnelServer_unregisterTunnel_Results_List(s *capnp.Segment, sz int32) (TunnelServer_unregisterTunnel_Results_List, error) (line 1645)
- (TunnelServer_unregisterTunnel_Results_List) At(i int) TunnelServer_unregisterTunnel_Results (line 1650)
- (TunnelServer_unregisterTunnel_Results_List) Set(i int, v TunnelServer_unregisterTunnel_Results) error (line 1654)
- (TunnelServer_unregisterTunnel_Results_List) String() string (line 1658)
- (TunnelServer_unregisterTunnel_Results_Promise) Struct() (TunnelServer_unregisterTunnel_Results, error) (line 1666)
- NewTunnelServer_obsoleteDeclarativeTunnelConnect_Params(s *capnp.Segment) (TunnelServer_obsoleteDeclarativeTunnelConnect_Params, error) (line 1676)
- NewRootTunnelServer_obsoleteDeclarativeTunnelConnect_Params(s *capnp.Segment) (TunnelServer_obsoleteDeclarativeTunnelConnect_Params, error) (line 1681)
- ReadRootTunnelServer_obsoleteDeclarativeTunnelConnect_Params(msg *capnp.Message) (TunnelServer_obsoleteDeclarativeTunnelConnect_Params, error) (line 1686)
- (TunnelServer_obsoleteDeclarativeTunnelConnect_Params) String() string (line 1691)
- NewTunnelServer_obsoleteDeclarativeTunnelConnect_Params_List(s *capnp.Segment, sz int32) (TunnelServer_obsoleteDeclarativeTunnelConnect_Params_List, error) (line 1700)
- (TunnelServer_obsoleteDeclarativeTunnelConnect_Params_List) At(i int) TunnelServer_obsoleteDeclarativeTunnelConnect_Params (line 1705)
- (TunnelServer_obsoleteDeclarativeTunnelConnect_Params_List) Set(i int, v TunnelServer_obsoleteDeclarativeTunnelConnect_Params) error (line 1709)
- (TunnelServer_obsoleteDeclarativeTunnelConnect_Params_List) String() string (line 1713)
- (TunnelServer_obsoleteDeclarativeTunnelConnect_Params_Promise) Struct() (TunnelServer_obsoleteDeclarativeTunnelConnect_Params, error) (line 1721)
- NewTunnelServer_obsoleteDeclarativeTunnelConnect_Results(s *capnp.Segment) (TunnelServer_obsoleteDeclarativeTunnelConnect_Results, error) (line 1731)
- NewRootTunnelServer_obsoleteDeclarativeTunnelConnect_Results(s *capnp.Segment) (TunnelServer_obsoleteDeclarativeTunnelConnect_Results, error) (line 1736)
- ReadRootTunnelServer_obsoleteDeclarativeTunnelConnect_Results(msg *capnp.Message) (TunnelServer_obsoleteDeclarativeTunnelConnect_Results, error) (line 1741)
- (TunnelServer_obsoleteDeclarativeTunnelConnect_Results) String() string (line 1746)
- NewTunnelServer_obsoleteDeclarativeTunnelConnect_Results_List(s *capnp.Segment, sz int32) (TunnelServer_obsoleteDeclarativeTunnelConnect_Results_List, error) (line 1755)
- (TunnelServer_obsoleteDeclarativeTunnelConnect_Results_List) At(i int) TunnelServer_obsoleteDeclarativeTunnelConnect_Results (line 1760)
- (TunnelServer_obsoleteDeclarativeTunnelConnect_Results_List) Set(i int, v TunnelServer_obsoleteDeclarativeTunnelConnect_Results) error (line 1764)
- (TunnelServer_obsoleteDeclarativeTunnelConnect_Results_List) String() string (line 1768)
- (TunnelServer_obsoleteDeclarativeTunnelConnect_Results_Promise) Struct() (TunnelServer_obsoleteDeclarativeTunnelConnect_Results, error) (line 1776)
- NewTunnelServer_authenticate_Params(s *capnp.Segment) (TunnelServer_authenticate_Params, error) (line 1786)
- NewRootTunnelServer_authenticate_Params(s *capnp.Segment) (TunnelServer_authenticate_Params, error) (line 1791)
- ReadRootTunnelServer_authenticate_Params(msg *capnp.Message) (TunnelServer_authenticate_Params, error) (line 1796)
- (TunnelServer_authenticate_Params) String() string (line 1801)
- (TunnelServer_authenticate_Params) OriginCert() ([]byte, error) (line 1806)
- (TunnelServer_authenticate_Params) HasOriginCert() bool (line 1811)
- (TunnelServer_authenticate_Params) SetOriginCert(v []byte) error (line 1816)
- (TunnelServer_authenticate_Params) Hostname() (string, error) (line 1820)
- (TunnelServer_authenticate_Params) HasHostname() bool (line 1825)
- (TunnelServer_authenticate_Params) HostnameBytes() ([]byte, error) (line 1830)
- (TunnelServer_authenticate_Params) SetHostname(v string) error (line 1835)
- (TunnelServer_authenticate_Params) Options() (RegistrationOptions, error) (line 1839)
- (TunnelServer_authenticate_Params) HasOptions() bool (line 1844)
- (TunnelServer_authenticate_Params) SetOptions(v RegistrationOptions) error (line 1849)
- (TunnelServer_authenticate_Params) NewOptions() (RegistrationOptions, error) (line 1855)
- NewTunnelServer_authenticate_Params_List(s *capnp.Segment, sz int32) (TunnelServer_authenticate_Params_List, error) (line 1868)
- (TunnelServer_authenticate_Params_List) At(i int) TunnelServer_authenticate_Params (line 1873)
- (TunnelServer_authenticate_Params_List) Set(i int, v TunnelServer_authenticate_Params) error (line 1877)
- (TunnelServer_authenticate_Params_List) String() string (line 1881)
- (TunnelServer_authenticate_Params_Promise) Struct() (TunnelServer_authenticate_Params, error) (line 1889)
- (TunnelServer_authenticate_Params_Promise) Options() RegistrationOptions_Promise (line 1894)
- NewTunnelServer_authenticate_Results(s *capnp.Segment) (TunnelServer_authenticate_Results, error) (line 1903)
- NewRootTunnelServer_authenticate_Results(s *capnp.Segment) (TunnelServer_authenticate_Results, error) (line 1908)
- ReadRootTunnelServer_authenticate_Results(msg *capnp.Message) (TunnelServer_authenticate_Results, error) (line 1913)
- (TunnelServer_authenticate_Results) String() string (line 1918)
- (TunnelServer_authenticate_Results) Result() (AuthenticateResponse, error) (line 1923)
- (TunnelServer_authenticate_Results) HasResult() bool (line 1928)
- (TunnelServer_authenticate_Results) SetResult(v AuthenticateResponse) error (line 1933)
- (TunnelServer_authenticate_Results) NewResult() (AuthenticateResponse, error) (line 1939)
- NewTunnelServer_authenticate_Results_List(s *capnp.Segment, sz int32) (TunnelServer_authenticate_Results_List, error) (line 1952)
- (TunnelServer_authenticate_Results_List) At(i int) TunnelServer_authenticate_Results (line 1957)
- (TunnelServer_authenticate_Results_List) Set(i int, v TunnelServer_authenticate_Results) error (line 1961)
- (TunnelServer_authenticate_Results_List) String() string (line 1965)
- (TunnelServer_authenticate_Results_Promise) Struct() (TunnelServer_authenticate_Results, error) (line 1973)
- (TunnelServer_authenticate_Results_Promise) Result() AuthenticateResponse_Promise (line 1978)
- NewTunnelServer_reconnectTunnel_Params(s *capnp.Segment) (TunnelServer_reconnectTunnel_Params, error) (line 1987)
- NewRootTunnelServer_reconnectTunnel_Params(s *capnp.Segment) (TunnelServer_reconnectTunnel_Params, error) (line 1992)
- ReadRootTunnelServer_reconnectTunnel_Params(msg *capnp.Message) (TunnelServer_reconnectTunnel_Params, error) (line 1997)
- (TunnelServer_reconnectTunnel_Params) String() string (line 2002)
- (TunnelServer_reconnectTunnel_Params) Jwt() ([]byte, error) (line 2007)
- (TunnelServer_reconnectTunnel_Params) HasJwt() bool (line 2012)
- (TunnelServer_reconnectTunnel_Params) SetJwt(v []byte) error (line 2017)
- (TunnelServer_reconnectTunnel_Params) EventDigest() ([]byte, error) (line 2021)
- (TunnelServer_reconnectTunnel_Params) HasEventDigest() bool (line 2026)
- (TunnelServer_reconnectTunnel_Params) SetEventDigest(v []byte) error (line 2031)
- (TunnelServer_reconnectTunnel_Params) ConnDigest() ([]byte, error) (line 2035)
- (TunnelServer_reconnectTunnel_Params) HasConnDigest() bool (line 2040)
- (TunnelServer_reconnectTunnel_Params) SetConnDigest(v []byte) error (line 2045)
- (TunnelServer_reconnectTunnel_Params) Hostname() (string, error) (line 2049)
- (TunnelServer_reconnectTunnel_Params) HasHostname() bool (line 2054)
- (TunnelServer_reconnectTunnel_Params) HostnameBytes() ([]byte, error) (line 2059)
- (TunnelServer_reconnectTunnel_Params) SetHostname(v string) error (line 2064)
- (TunnelServer_reconnectTunnel_Params) Options() (RegistrationOptions, error) (line 2068)
- (TunnelServer_reconnectTunnel_Params) HasOptions() bool (line 2073)
- (TunnelServer_reconnectTunnel_Params) SetOptions(v RegistrationOptions) error (line 2078)
- (TunnelServer_reconnectTunnel_Params) NewOptions() (RegistrationOptions, error) (line 2084)
- NewTunnelServer_reconnectTunnel_Params_List(s *capnp.Segment, sz int32) (TunnelServer_reconnectTunnel_Params_List, error) (line 2097)
- (TunnelServer_reconnectTunnel_Params_List) At(i int) TunnelServer_reconnectTunnel_Params (line 2102)
- (TunnelServer_reconnectTunnel_Params_List) Set(i int, v TunnelServer_reconnectTunnel_Params) error (line 2106)
- (TunnelServer_reconnectTunnel_Params_List) String() string (line 2110)
- (TunnelServer_reconnectTunnel_Params_Promise) Struct() (TunnelServer_reconnectTunnel_Params, error) (line 2118)
- (TunnelServer_reconnectTunnel_Params_Promise) Options() RegistrationOptions_Promise (line 2123)
- NewTunnelServer_reconnectTunnel_Results(s *capnp.Segment) (TunnelServer_reconnectTunnel_Results, error) (line 2132)
- NewRootTunnelServer_reconnectTunnel_Results(s *capnp.Segment) (TunnelServer_reconnectTunnel_Results, error) (line 2137)
- ReadRootTunnelServer_reconnectTunnel_Results(msg *capnp.Message) (TunnelServer_reconnectTunnel_Results, error) (line 2142)
- (TunnelServer_reconnectTunnel_Results) String() string (line 2147)
- (TunnelServer_reconnectTunnel_Results) Result() (TunnelRegistration, error) (line 2152)
- (TunnelServer_reconnectTunnel_Results) HasResult() bool (line 2157)
- (TunnelServer_reconnectTunnel_Results) SetResult(v TunnelRegistration) error (line 2162)
- (TunnelServer_reconnectTunnel_Results) NewResult() (TunnelRegistration, error) (line 2168)
- NewTunnelServer_reconnectTunnel_Results_List(s *capnp.Segment, sz int32) (TunnelServer_reconnectTunnel_Results_List, error) (line 2181)
- (TunnelServer_reconnectTunnel_Results_List) At(i int) TunnelServer_reconnectTunnel_Results (line 2186)
- (TunnelServer_reconnectTunnel_Results_List) Set(i int, v TunnelServer_reconnectTunnel_Results) error (line 2190)
- (TunnelServer_reconnectTunnel_Results_List) String() string (line 2194)
- (TunnelServer_reconnectTunnel_Results_Promise) Struct() (TunnelServer_reconnectTunnel_Results, error) (line 2202)
- (TunnelServer_reconnectTunnel_Results_Promise) Result() TunnelRegistration_Promise (line 2207)
- NewTag(s *capnp.Segment) (Tag, error) (line 2216)
- NewRootTag(s *capnp.Segment) (Tag, error) (line 2221)
- ReadRootTag(msg *capnp.Message) (Tag, error) (line 2226)
- (Tag) String() string (line 2231)
- (Tag) Name() (string, error) (line 2236)
- (Tag) HasName() bool (line 2241)
- (Tag) NameBytes() ([]byte, error) (line 2246)
- (Tag) SetName(v string) error (line 2251)
- (Tag) Value() (string, error) (line 2255)
- (Tag) HasValue() bool (line 2260)
- (Tag) ValueBytes() ([]byte, error) (line 2265)
- (Tag) SetValue(v string) error (line 2270)
- NewTag_List(s *capnp.Segment, sz int32) (Tag_List, error) (line 2278)
- (Tag_List) At(i int) Tag (line 2283)
- (Tag_List) Set(i int, v Tag) error (line 2285)
- (Tag_List) String() string (line 2287)
- (Tag_Promise) Struct() (Tag, error) (line 2295)
- NewClientInfo(s *capnp.Segment) (ClientInfo, error) (line 2305)
- NewRootClientInfo(s *capnp.Segment) (ClientInfo, error) (line 2310)
- ReadRootClientInfo(msg *capnp.Message) (ClientInfo, error) (line 2315)
- (ClientInfo) String() string (line 2320)
- (ClientInfo) ClientId() ([]byte, error) (line 2325)
- (ClientInfo) HasClientId() bool (line 2330)
- (ClientInfo) SetClientId(v []byte) error (line 2335)
- (ClientInfo) Features() (capnp.TextList, error) (line 2339)
- (ClientInfo) HasFeatures() bool (line 2344)
- (ClientInfo) SetFeatures(v capnp.TextList) error (line 2349)
- (ClientInfo) NewFeatures(n int32) (capnp.TextList, error) (line 2355)
- (ClientInfo) Version() (string, error) (line 2364)
- (ClientInfo) HasVersion() bool (line 2369)
- (ClientInfo) VersionBytes() ([]byte, error) (line 2374)
- (ClientInfo) SetVersion(v string) error (line 2379)
- (ClientInfo) Arch() (string, error) (line 2383)
- (ClientInfo) HasArch() bool (line 2388)
- (ClientInfo) ArchBytes() ([]byte, error) (line 2393)
- (ClientInfo) SetArch(v string) error (line 2398)
- NewClientInfo_List(s *capnp.Segment, sz int32) (ClientInfo_List, error) (line 2406)
- (ClientInfo_List) At(i int) ClientInfo (line 2411)
- (ClientInfo_List) Set(i int, v ClientInfo) error (line 2413)
- (ClientInfo_List) String() string (line 2415)
- (ClientInfo_Promise) Struct() (ClientInfo, error) (line 2423)
- NewConnectionOptions(s *capnp.Segment) (ConnectionOptions, error) (line 2433)
- NewRootConnectionOptions(s *capnp.Segment) (ConnectionOptions, error) (line 2438)
- ReadRootConnectionOptions(msg *capnp.Message) (ConnectionOptions, error) (line 2443)
- (ConnectionOptions) String() string (line 2448)
- (ConnectionOptions) Client() (ClientInfo, error) (line 2453)
- (ConnectionOptions) HasClient() bool (line 2458)
- (ConnectionOptions) SetClient(v ClientInfo) error (line 2463)
- (ConnectionOptions) NewClient() (ClientInfo, error) (line 2469)
- (ConnectionOptions) OriginLocalIp() ([]byte, error) (line 2478)
- (ConnectionOptions) HasOriginLocalIp() bool (line 2483)
- (ConnectionOptions) SetOriginLocalIp(v []byte) error (line 2488)
- (ConnectionOptions) ReplaceExisting() bool (line 2492)
- (ConnectionOptions) SetReplaceExisting(v bool) (line 2496)
- (ConnectionOptions) CompressionQuality() uint8 (line 2500)
- (ConnectionOptions) SetCompressionQuality(v uint8) (line 2504)
- (ConnectionOptions) NumPreviousAttempts() uint8 (line 2508)
- (ConnectionOptions) SetNumPreviousAttempts(v uint8) (line 2512)
- NewConnectionOptions_List(s *capnp.Segment, sz int32) (ConnectionOptions_List, error) (line 2520)
- (ConnectionOptions_List) At(i int) ConnectionOptions (line 2525)
- (ConnectionOptions_List) Set(i int, v ConnectionOptions) error (line 2529)
- (ConnectionOptions_List) String() string (line 2533)
- (ConnectionOptions_Promise) Struct() (ConnectionOptions, error) (line 2541)
- (ConnectionOptions_Promise) Client() ClientInfo_Promise (line 2546)
- (ConnectionResponse_result_Which) String() string (line 2559)
- NewConnectionResponse(s *capnp.Segment) (ConnectionResponse, error) (line 2574)
- NewRootConnectionResponse(s *capnp.Segment) (ConnectionResponse, error) (line 2579)
- ReadRootConnectionResponse(msg *capnp.Message) (ConnectionResponse, error) (line 2584)
- (ConnectionResponse) String() string (line 2589)
- (ConnectionResponse) Result() ConnectionResponse_result (line 2594)
- (ConnectionResponse_result) Which() ConnectionResponse_result_Which (line 2596)
- (ConnectionResponse_result) Error() (ConnectionError, error) (line 2599)
- (ConnectionResponse_result) HasError() bool (line 2607)
- (ConnectionResponse_result) SetError(v ConnectionError) error (line 2615)
- (ConnectionResponse_result) NewError() (ConnectionError, error) (line 2622)
- (ConnectionResponse_result) ConnectionDetails() (ConnectionDetails, error) (line 2632)
- (ConnectionResponse_result) HasConnectionDetails() bool (line 2640)
- (ConnectionResponse_result) SetConnectionDetails(v ConnectionDetails) error (line 2648)
- (ConnectionResponse_result) NewConnectionDetails() (ConnectionDetails, error) (line 2655)
- NewConnectionResponse_List(s *capnp.Segment, sz int32) (ConnectionResponse_List, error) (line 2669)
- (ConnectionResponse_List) At(i int) ConnectionResponse (line 2674)
- (ConnectionResponse_List) Set(i int, v ConnectionResponse) error (line 2678)
- (ConnectionResponse_List) String() string (line 2682)
- (ConnectionResponse_Promise) Struct() (ConnectionResponse, error) (line 2690)
- (ConnectionResponse_Promise) Result() ConnectionResponse_result_Promise (line 2695)
- (ConnectionResponse_result_Promise) Struct() (ConnectionResponse_result, error) (line 2702)
- (ConnectionResponse_result_Promise) Error() ConnectionError_Promise (line 2707)
- (ConnectionResponse_result_Promise) ConnectionDetails() ConnectionDetails_Promise (line 2711)
- NewConnectionError(s *capnp.Segment) (ConnectionError, error) (line 2720)
- NewRootConnectionError(s *capnp.Segment) (ConnectionError, error) (line 2725)
- ReadRootConnectionError(msg *capnp.Message) (ConnectionError, error) (line 2730)
- (ConnectionError) String() string (line 2735)
- (ConnectionError) Cause() (string, error) (line 2740)
- (ConnectionError) HasCause() bool (line 2745)
- (ConnectionError) CauseBytes() ([]byte, error) (line 2750)
- (ConnectionError) SetCause(v string) error (line 2755)
- (ConnectionError) RetryAfter() int64 (line 2759)
- (ConnectionError) SetRetryAfter(v int64) (line 2763)
- (ConnectionError) ShouldRetry() bool (line 2767)
- (ConnectionError) SetShouldRetry(v bool) (line 2771)
- NewConnectionError_List(s *capnp.Segment, sz int32) (ConnectionError_List, error) (line 2779)
- (ConnectionError_List) At(i int) ConnectionError (line 2784)
- (ConnectionError_List) Set(i int, v ConnectionError) error (line 2786)
- (ConnectionError_List) String() string (line 2790)
- (ConnectionError_Promise) Struct() (ConnectionError, error) (line 2798)
- NewConnectionDetails(s *capnp.Segment) (ConnectionDetails, error) (line 2808)
- NewRootConnectionDetails(s *capnp.Segment) (ConnectionDetails, error) (line 2813)
- ReadRootConnectionDetails(msg *capnp.Message) (ConnectionDetails, error) (line 2818)
- (ConnectionDetails) String() string (line 2823)
- (ConnectionDetails) Uuid() ([]byte, error) (line 2828)
- (ConnectionDetails) HasUuid() bool (line 2833)
- (ConnectionDetails) SetUuid(v []byte) error (line 2838)
- (ConnectionDetails) LocationName() (string, error) (line 2842)
- (ConnectionDetails) HasLocationName() bool (line 2847)
- (ConnectionDetails) LocationNameBytes() ([]byte, error) (line 2852)
- (ConnectionDetails) SetLocationName(v string) error (line 2857)
- (ConnectionDetails) TunnelIsRemotelyManaged() bool (line 2861)
- (ConnectionDetails) SetTunnelIsRemotelyManaged(v bool) (line 2865)
- NewConnectionDetails_List(s *capnp.Segment, sz int32) (ConnectionDetails_List, error) (line 2873)
- (ConnectionDetails_List) At(i int) ConnectionDetails (line 2878)
- (ConnectionDetails_List) Set(i int, v ConnectionDetails) error (line 2882)
- (ConnectionDetails_List) String() string (line 2886)
- (ConnectionDetails_Promise) Struct() (ConnectionDetails, error) (line 2894)
- NewTunnelAuth(s *capnp.Segment) (TunnelAuth, error) (line 2904)
- NewRootTunnelAuth(s *capnp.Segment) (TunnelAuth, error) (line 2909)
- ReadRootTunnelAuth(msg *capnp.Message) (TunnelAuth, error) (line 2914)
- (TunnelAuth) String() string (line 2919)
- (TunnelAuth) AccountTag() (string, error) (line 2924)
- (TunnelAuth) HasAccountTag() bool (line 2929)
- (TunnelAuth) AccountTagBytes() ([]byte, error) (line 2934)
- (TunnelAuth) SetAccountTag(v string) error (line 2939)
- (TunnelAuth) TunnelSecret() ([]byte, error) (line 2943)
- (TunnelAuth) HasTunnelSecret() bool (line 2948)
- (TunnelAuth) SetTunnelSecret(v []byte) error (line 2953)
- NewTunnelAuth_List(s *capnp.Segment, sz int32) (TunnelAuth_List, error) (line 2961)
- (TunnelAuth_List) At(i int) TunnelAuth (line 2966)
- (TunnelAuth_List) Set(i int, v TunnelAuth) error (line 2968)
- (TunnelAuth_List) String() string (line 2970)
- (TunnelAuth_Promise) Struct() (TunnelAuth, error) (line 2978)
- (RegistrationServer) RegisterConnection(ctx context.Context, params func(RegistrationServer_registerConnection_Params) error, opts ...capnp.CallOption) RegistrationServer_registerConnection_Results_Promise (line 2988)
- (RegistrationServer) UnregisterConnection(ctx context.Context, params func(RegistrationServer_unregisterConnection_Params) error, opts ...capnp.CallOption) RegistrationServer_unregisterConnection_Results_Promise (line 3008)
- (RegistrationServer) UpdateLocalConfiguration(ctx context.Context, params func(RegistrationServer_updateLocalConfiguration_Params) error, opts ...capnp.CallOption) RegistrationServer_updateLocalConfiguration_Results_Promise (line 3028)
- RegistrationServer_ServerToClient(s RegistrationServer_Server) RegistrationServer (line 3059)
- RegistrationServer_Methods(methods []server.Method, s RegistrationServer_Server) []server.Method (line 3064)
- NewRegistrationServer_registerConnection_Params(s *capnp.Segment) (RegistrationServer_registerConnection_Params, error) (line 3143)
- NewRootRegistrationServer_registerConnection_Params(s *capnp.Segment) (RegistrationServer_registerConnection_Params, error) (line 3148)
- ReadRootRegistrationServer_registerConnection_Params(msg *capnp.Message) (RegistrationServer_registerConnection_Params, error) (line 3153)
- (RegistrationServer_registerConnection_Params) String() string (line 3158)
- (RegistrationServer_registerConnection_Params) Auth() (TunnelAuth, error) (line 3163)
- (RegistrationServer_registerConnection_Params) HasAuth() bool (line 3168)
- (RegistrationServer_registerConnection_Params) SetAuth(v TunnelAuth) error (line 3173)
- (RegistrationServer_registerConnection_Params) NewAuth() (TunnelAuth, error) (line 3179)
- (RegistrationServer_registerConnection_Params) TunnelId() ([]byte, error) (line 3188)
- (RegistrationServer_registerConnection_Params) HasTunnelId() bool (line 3193)
- (RegistrationServer_registerConnection_Params) SetTunnelId(v []byte) error (line 3198)
- (RegistrationServer_registerConnection_Params) ConnIndex() uint8 (line 3202)
- (RegistrationServer_registerConnection_Params) SetConnIndex(v uint8) (line 3206)
- (RegistrationServer_registerConnection_Params) Options() (ConnectionOptions, error) (line 3210)
- (RegistrationServer_registerConnection_Params) HasOptions() bool (line 3215)
- (RegistrationServer_registerConnection_Params) SetOptions(v ConnectionOptions) error (line 3220)
- (RegistrationServer_registerConnection_Params) NewOptions() (ConnectionOptions, error) (line 3226)
- NewRegistrationServer_registerConnection_Params_List(s *capnp.Segment, sz int32) (RegistrationServer_registerConnection_Params_List, error) (line 3239)
- (RegistrationServer_registerConnection_Params_List) At(i int) RegistrationServer_registerConnection_Params (line 3244)
- (RegistrationServer_registerConnection_Params_List) Set(i int, v RegistrationServer_registerConnection_Params) error (line 3248)
- (RegistrationServer_registerConnection_Params_List) String() string (line 3252)
- (RegistrationServer_registerConnection_Params_Promise) Struct() (RegistrationServer_registerConnection_Params, error) (line 3260)
- (RegistrationServer_registerConnection_Params_Promise) Auth() TunnelAuth_Promise (line 3265)
- (RegistrationServer_registerConnection_Params_Promise) Options() ConnectionOptions_Promise (line 3269)
- NewRegistrationServer_registerConnection_Results(s *capnp.Segment) (RegistrationServer_registerConnection_Results, error) (line 3278)
- NewRootRegistrationServer_registerConnection_Results(s *capnp.Segment) (RegistrationServer_registerConnection_Results, error) (line 3283)
- ReadRootRegistrationServer_registerConnection_Results(msg *capnp.Message) (RegistrationServer_registerConnection_Results, error) (line 3288)
- (RegistrationServer_registerConnection_Results) String() string (line 3293)
- (RegistrationServer_registerConnection_Results) Result() (ConnectionResponse, error) (line 3298)
- (RegistrationServer_registerConnection_Results) HasResult() bool (line 3303)
- (RegistrationServer_registerConnection_Results) SetResult(v ConnectionResponse) error (line 3308)
- (RegistrationServer_registerConnection_Results) NewResult() (ConnectionResponse, error) (line 3314)
- NewRegistrationServer_registerConnection_Results_List(s *capnp.Segment, sz int32) (RegistrationServer_registerConnection_Results_List, error) (line 3327)
- (RegistrationServer_registerConnection_Results_List) At(i int) RegistrationServer_registerConnection_Results (line 3332)
- (RegistrationServer_registerConnection_Results_List) Set(i int, v RegistrationServer_registerConnection_Results) error (line 3336)
- (RegistrationServer_registerConnection_Results_List) String() string (line 3340)
- (RegistrationServer_registerConnection_Results_Promise) Struct() (RegistrationServer_registerConnection_Results, error) (line 3348)
- (RegistrationServer_registerConnection_Results_Promise) Result() ConnectionResponse_Promise (line 3353)
- NewRegistrationServer_unregisterConnection_Params(s *capnp.Segment) (RegistrationServer_unregisterConnection_Params, error) (line 3362)
- NewRootRegistrationServer_unregisterConnection_Params(s *capnp.Segment) (RegistrationServer_unregisterConnection_Params, error) (line 3367)
- ReadRootRegistrationServer_unregisterConnection_Params(msg *capnp.Message) (RegistrationServer_unregisterConnection_Params, error) (line 3372)
- (RegistrationServer_unregisterConnection_Params) String() string (line 3377)
- NewRegistrationServer_unregisterConnection_Params_List(s *capnp.Segment, sz int32) (RegistrationServer_unregisterConnection_Params_List, error) (line 3386)
- (RegistrationServer_unregisterConnection_Params_List) At(i int) RegistrationServer_unregisterConnection_Params (line 3391)
- (RegistrationServer_unregisterConnection_Params_List) Set(i int, v RegistrationServer_unregisterConnection_Params) error (line 3395)
- (RegistrationServer_unregisterConnection_Params_List) String() string (line 3399)
- (RegistrationServer_unregisterConnection_Params_Promise) Struct() (RegistrationServer_unregisterConnection_Params, error) (line 3407)
- NewRegistrationServer_unregisterConnection_Results(s *capnp.Segment) (RegistrationServer_unregisterConnection_Results, error) (line 3417)
- NewRootRegistrationServer_unregisterConnection_Results(s *capnp.Segment) (RegistrationServer_unregisterConnection_Results, error) (line 3422)
- ReadRootRegistrationServer_unregisterConnection_Results(msg *capnp.Message) (RegistrationServer_unregisterConnection_Results, error) (line 3427)
- (RegistrationServer_unregisterConnection_Results) String() string (line 3432)
- NewRegistrationServer_unregisterConnection_Results_List(s *capnp.Segment, sz int32) (RegistrationServer_unregisterConnection_Results_List, error) (line 3441)
- (RegistrationServer_unregisterConnection_Results_List) At(i int) RegistrationServer_unregisterConnection_Results (line 3446)
- (RegistrationServer_unregisterConnection_Results_List) Set(i int, v RegistrationServer_unregisterConnection_Results) error (line 3450)
- (RegistrationServer_unregisterConnection_Results_List) String() string (line 3454)
- (RegistrationServer_unregisterConnection_Results_Promise) Struct() (RegistrationServer_unregisterConnection_Results, error) (line 3462)
- NewRegistrationServer_updateLocalConfiguration_Params(s *capnp.Segment) (RegistrationServer_updateLocalConfiguration_Params, error) (line 3472)
- NewRootRegistrationServer_updateLocalConfiguration_Params(s *capnp.Segment) (RegistrationServer_updateLocalConfiguration_Params, error) (line 3477)
- ReadRootRegistrationServer_updateLocalConfiguration_Params(msg *capnp.Message) (RegistrationServer_updateLocalConfiguration_Params, error) (line 3482)
- (RegistrationServer_updateLocalConfiguration_Params) String() string (line 3487)
- (RegistrationServer_updateLocalConfiguration_Params) Config() ([]byte, error) (line 3492)
- (RegistrationServer_updateLocalConfiguration_Params) HasConfig() bool (line 3497)
- (RegistrationServer_updateLocalConfiguration_Params) SetConfig(v []byte) error (line 3502)
- NewRegistrationServer_updateLocalConfiguration_Params_List(s *capnp.Segment, sz int32) (RegistrationServer_updateLocalConfiguration_Params_List, error) (line 3510)
- (RegistrationServer_updateLocalConfiguration_Params_List) At(i int) RegistrationServer_updateLocalConfiguration_Params (line 3515)
- (RegistrationServer_updateLocalConfiguration_Params_List) Set(i int, v RegistrationServer_updateLocalConfiguration_Params) error (line 3519)
- (RegistrationServer_updateLocalConfiguration_Params_List) String() string (line 3523)
- (RegistrationServer_updateLocalConfiguration_Params_Promise) Struct() (RegistrationServer_updateLocalConfiguration_Params, error) (line 3531)
- NewRegistrationServer_updateLocalConfiguration_Results(s *capnp.Segment) (RegistrationServer_updateLocalConfiguration_Results, error) (line 3541)
- NewRootRegistrationServer_updateLocalConfiguration_Results(s *capnp.Segment) (RegistrationServer_updateLocalConfiguration_Results, error) (line 3546)
- ReadRootRegistrationServer_updateLocalConfiguration_Results(msg *capnp.Message) (RegistrationServer_updateLocalConfiguration_Results, error) (line 3551)
- (RegistrationServer_updateLocalConfiguration_Results) String() string (line 3556)
- NewRegistrationServer_updateLocalConfiguration_Results_List(s *capnp.Segment, sz int32) (RegistrationServer_updateLocalConfiguration_Results_List, error) (line 3565)
- (RegistrationServer_updateLocalConfiguration_Results_List) At(i int) RegistrationServer_updateLocalConfiguration_Results (line 3570)
- (RegistrationServer_updateLocalConfiguration_Results_List) Set(i int, v RegistrationServer_updateLocalConfiguration_Results) error (line 3574)
- (RegistrationServer_updateLocalConfiguration_Results_List) String() string (line 3578)
- (RegistrationServer_updateLocalConfiguration_Results_Promise) Struct() (RegistrationServer_updateLocalConfiguration_Results, error) (line 3586)
- NewRegisterUdpSessionResponse(s *capnp.Segment) (RegisterUdpSessionResponse, error) (line 3596)
- NewRootRegisterUdpSessionResponse(s *capnp.Segment) (RegisterUdpSessionResponse, error) (line 3601)
- ReadRootRegisterUdpSessionResponse(msg *capnp.Message) (RegisterUdpSessionResponse, error) (line 3606)
- (RegisterUdpSessionResponse) String() string (line 3611)
- (RegisterUdpSessionResponse) Err() (string, error) (line 3616)
- (RegisterUdpSessionResponse) HasErr() bool (line 3621)
- (RegisterUdpSessionResponse) ErrBytes() ([]byte, error) (line 3626)
- (RegisterUdpSessionResponse) SetErr(v string) error (line 3631)
- (RegisterUdpSessionResponse) Spans() ([]byte, error) (line 3635)
- (RegisterUdpSessionResponse) HasSpans() bool (line 3640)
- (RegisterUdpSessionResponse) SetSpans(v []byte) error (line 3645)
- NewRegisterUdpSessionResponse_List(s *capnp.Segment, sz int32) (RegisterUdpSessionResponse_List, error) (line 3653)
- (RegisterUdpSessionResponse_List) At(i int) RegisterUdpSessionResponse (line 3658)
- (RegisterUdpSessionResponse_List) Set(i int, v RegisterUdpSessionResponse) error (line 3662)
- (RegisterUdpSessionResponse_List) String() string (line 3666)
- (RegisterUdpSessionResponse_Promise) Struct() (RegisterUdpSessionResponse, error) (line 3674)
- (SessionManager) RegisterUdpSession(ctx context.Context, params func(SessionManager_registerUdpSession_Params) error, opts ...capnp.CallOption) SessionManager_registerUdpSession_Results_Promise (line 3684)
- (SessionManager) UnregisterUdpSession(ctx context.Context, params func(SessionManager_unregisterUdpSession_Params) error, opts ...capnp.CallOption) SessionManager_unregisterUdpSession_Results_Promise (line 3704)
- SessionManager_ServerToClient(s SessionManager_Server) SessionManager (line 3731)
- SessionManager_Methods(methods []server.Method, s SessionManager_Server) []server.Method (line 3736)
- NewSessionManager_registerUdpSession_Params(s *capnp.Segment) (SessionManager_registerUdpSession_Params, error) (line 3793)
- NewRootSessionManager_registerUdpSession_Params(s *capnp.Segment) (SessionManager_registerUdpSession_Params, error) (line 3798)
- ReadRootSessionManager_registerUdpSession_Params(msg *capnp.Message) (SessionManager_registerUdpSession_Params, error) (line 3803)
- (SessionManager_registerUdpSession_Params) String() string (line 3808)
- (SessionManager_registerUdpSession_Params) SessionId() ([]byte, error) (line 3813)
- (SessionManager_registerUdpSession_Params) HasSessionId() bool (line 3818)
- (SessionManager_registerUdpSession_Params) SetSessionId(v []byte) error (line 3823)
- (SessionManager_registerUdpSession_Params) DstIp() ([]byte, error) (line 3827)
- (SessionManager_registerUdpSession_Params) HasDstIp() bool (line 3832)
- (SessionManager_registerUdpSession_Params) SetDstIp(v []byte) error (line 3837)
- (SessionManager_registerUdpSession_Params) DstPort() uint16 (line 3841)
- (SessionManager_registerUdpSession_Params) SetDstPort(v uint16) (line 3845)
- (SessionManager_registerUdpSession_Params) CloseAfterIdleHint() int64 (line 3849)
- (SessionManager_registerUdpSession_Params) SetCloseAfterIdleHint(v int64) (line 3853)
- (SessionManager_registerUdpSession_Params) TraceContext() (string, error) (line 3857)
- (SessionManager_registerUdpSession_Params) HasTraceContext() bool (line 3862)
- (SessionManager_registerUdpSession_Params) TraceContextBytes() ([]byte, error) (line 3867)
- (SessionManager_registerUdpSession_Params) SetTraceContext(v string) error (line 3872)
- NewSessionManager_registerUdpSession_Params_List(s *capnp.Segment, sz int32) (SessionManager_registerUdpSession_Params_List, error) (line 3880)
- (SessionManager_registerUdpSession_Params_List) At(i int) SessionManager_registerUdpSession_Params (line 3885)
- (SessionManager_registerUdpSession_Params_List) Set(i int, v SessionManager_registerUdpSession_Params) error (line 3889)
- (SessionManager_registerUdpSession_Params_List) String() string (line 3893)
- (SessionManager_registerUdpSession_Params_Promise) Struct() (SessionManager_registerUdpSession_Params, error) (line 3901)
- NewSessionManager_registerUdpSession_Results(s *capnp.Segment) (SessionManager_registerUdpSession_Results, error) (line 3911)
- NewRootSessionManager_registerUdpSession_Results(s *capnp.Segment) (SessionManager_registerUdpSession_Results, error) (line 3916)
- ReadRootSessionManager_registerUdpSession_Results(msg *capnp.Message) (SessionManager_registerUdpSession_Results, error) (line 3921)
- (SessionManager_registerUdpSession_Results) String() string (line 3926)
- (SessionManager_registerUdpSession_Results) Result() (RegisterUdpSessionResponse, error) (line 3931)
- (SessionManager_registerUdpSession_Results) HasResult() bool (line 3936)
- (SessionManager_registerUdpSession_Results) SetResult(v RegisterUdpSessionResponse) error (line 3941)
- (SessionManager_registerUdpSession_Results) NewResult() (RegisterUdpSessionResponse, error) (line 3947)
- NewSessionManager_registerUdpSession_Results_List(s *capnp.Segment, sz int32) (SessionManager_registerUdpSession_Results_List, error) (line 3960)
- (SessionManager_registerUdpSession_Results_List) At(i int) SessionManager_registerUdpSession_Results (line 3965)
- (SessionManager_registerUdpSession_Results_List) Set(i int, v SessionManager_registerUdpSession_Results) error (line 3969)
- (SessionManager_registerUdpSession_Results_List) String() string (line 3973)
- (SessionManager_registerUdpSession_Results_Promise) Struct() (SessionManager_registerUdpSession_Results, error) (line 3981)
- (SessionManager_registerUdpSession_Results_Promise) Result() RegisterUdpSessionResponse_Promise (line 3986)
- NewSessionManager_unregisterUdpSession_Params(s *capnp.Segment) (SessionManager_unregisterUdpSession_Params, error) (line 3995)
- NewRootSessionManager_unregisterUdpSession_Params(s *capnp.Segment) (SessionManager_unregisterUdpSession_Params, error) (line 4000)
- ReadRootSessionManager_unregisterUdpSession_Params(msg *capnp.Message) (SessionManager_unregisterUdpSession_Params, error) (line 4005)
- (SessionManager_unregisterUdpSession_Params) String() string (line 4010)
- (SessionManager_unregisterUdpSession_Params) SessionId() ([]byte, error) (line 4015)
- (SessionManager_unregisterUdpSession_Params) HasSessionId() bool (line 4020)
- (SessionManager_unregisterUdpSession_Params) SetSessionId(v []byte) error (line 4025)
- (SessionManager_unregisterUdpSession_Params) Message() (string, error) (line 4029)
- (SessionManager_unregisterUdpSession_Params) HasMessage() bool (line 4034)
- (SessionManager_unregisterUdpSession_Params) MessageBytes() ([]byte, error) (line 4039)
- (SessionManager_unregisterUdpSession_Params) SetMessage(v string) error (line 4044)
- NewSessionManager_unregisterUdpSession_Params_List(s *capnp.Segment, sz int32) (SessionManager_unregisterUdpSession_Params_List, error) (line 4052)
- (SessionManager_unregisterUdpSession_Params_List) At(i int) SessionManager_unregisterUdpSession_Params (line 4057)
- (SessionManager_unregisterUdpSession_Params_List) Set(i int, v SessionManager_unregisterUdpSession_Params) error (line 4061)
- (SessionManager_unregisterUdpSession_Params_List) String() string (line 4065)
- (SessionManager_unregisterUdpSession_Params_Promise) Struct() (SessionManager_unregisterUdpSession_Params, error) (line 4073)
- NewSessionManager_unregisterUdpSession_Results(s *capnp.Segment) (SessionManager_unregisterUdpSession_Results, error) (line 4083)
- NewRootSessionManager_unregisterUdpSession_Results(s *capnp.Segment) (SessionManager_unregisterUdpSession_Results, error) (line 4088)
- ReadRootSessionManager_unregisterUdpSession_Results(msg *capnp.Message) (SessionManager_unregisterUdpSession_Results, error) (line 4093)
- (SessionManager_unregisterUdpSession_Results) String() string (line 4098)
- NewSessionManager_unregisterUdpSession_Results_List(s *capnp.Segment, sz int32) (SessionManager_unregisterUdpSession_Results_List, error) (line 4107)
- (SessionManager_unregisterUdpSession_Results_List) At(i int) SessionManager_unregisterUdpSession_Results (line 4112)
- (SessionManager_unregisterUdpSession_Results_List) Set(i int, v SessionManager_unregisterUdpSession_Results) error (line 4116)
- (SessionManager_unregisterUdpSession_Results_List) String() string (line 4120)
- (SessionManager_unregisterUdpSession_Results_Promise) Struct() (SessionManager_unregisterUdpSession_Results, error) (line 4128)
- NewUpdateConfigurationResponse(s *capnp.Segment) (UpdateConfigurationResponse, error) (line 4138)
- NewRootUpdateConfigurationResponse(s *capnp.Segment) (UpdateConfigurationResponse, error) (line 4143)
- ReadRootUpdateConfigurationResponse(msg *capnp.Message) (UpdateConfigurationResponse, error) (line 4148)
- (UpdateConfigurationResponse) String() string (line 4153)
- (UpdateConfigurationResponse) LatestAppliedVersion() int32 (line 4158)
- (UpdateConfigurationResponse) SetLatestAppliedVersion(v int32) (line 4162)
- (UpdateConfigurationResponse) Err() (string, error) (line 4166)
- (UpdateConfigurationResponse) HasErr() bool (line 4171)
- (UpdateConfigurationResponse) ErrBytes() ([]byte, error) (line 4176)
- (UpdateConfigurationResponse) SetErr(v string) error (line 4181)
- NewUpdateConfigurationResponse_List(s *capnp.Segment, sz int32) (UpdateConfigurationResponse_List, error) (line 4189)
- (UpdateConfigurationResponse_List) At(i int) UpdateConfigurationResponse (line 4194)
- (UpdateConfigurationResponse_List) Set(i int, v UpdateConfigurationResponse) error (line 4198)
- (UpdateConfigurationResponse_List) String() string (line 4202)
- (UpdateConfigurationResponse_Promise) Struct() (UpdateConfigurationResponse, error) (line 4210)
- (ConfigurationManager) UpdateConfiguration(ctx context.Context, params func(ConfigurationManager_updateConfiguration_Params) error, opts ...capnp.CallOption) ConfigurationManager_updateConfiguration_Results_Promise (line 4220)
- ConfigurationManager_ServerToClient(s ConfigurationManager_Server) ConfigurationManager (line 4245)
- ConfigurationManager_Methods(methods []server.Method, s ConfigurationManager_Server) []server.Method (line 4250)
- NewConfigurationManager_updateConfiguration_Params(s *capnp.Segment) (ConfigurationManager_updateConfiguration_Params, error) (line 4285)
- NewRootConfigurationManager_updateConfiguration_Params(s *capnp.Segment) (ConfigurationManager_updateConfiguration_Params, error) (line 4290)
- ReadRootConfigurationManager_updateConfiguration_Params(msg *capnp.Message) (ConfigurationManager_updateConfiguration_Params, error) (line 4295)
- (ConfigurationManager_updateConfiguration_Params) String() string (line 4300)
- (ConfigurationManager_updateConfiguration_Params) Version() int32 (line 4305)
- (ConfigurationManager_updateConfiguration_Params) SetVersion(v int32) (line 4309)
- (ConfigurationManager_updateConfiguration_Params) Config() ([]byte, error) (line 4313)
- (ConfigurationManager_updateConfiguration_Params) HasConfig() bool (line 4318)
- (ConfigurationManager_updateConfiguration_Params) SetConfig(v []byte) error (line 4323)
- NewConfigurationManager_updateConfiguration_Params_List(s *capnp.Segment, sz int32) (ConfigurationManager_updateConfiguration_Params_List, error) (line 4331)
- (ConfigurationManager_updateConfiguration_Params_List) At(i int) ConfigurationManager_updateConfiguration_Params (line 4336)
- (ConfigurationManager_updateConfiguration_Params_List) Set(i int, v ConfigurationManager_updateConfiguration_Params) error (line 4340)
- (ConfigurationManager_updateConfiguration_Params_List) String() string (line 4344)
- (ConfigurationManager_updateConfiguration_Params_Promise) Struct() (ConfigurationManager_updateConfiguration_Params, error) (line 4352)
- NewConfigurationManager_updateConfiguration_Results(s *capnp.Segment) (ConfigurationManager_updateConfiguration_Results, error) (line 4362)
- NewRootConfigurationManager_updateConfiguration_Results(s *capnp.Segment) (ConfigurationManager_updateConfiguration_Results, error) (line 4367)
- ReadRootConfigurationManager_updateConfiguration_Results(msg *capnp.Message) (ConfigurationManager_updateConfiguration_Results, error) (line 4372)
- (ConfigurationManager_updateConfiguration_Results) String() string (line 4377)
- (ConfigurationManager_updateConfiguration_Results) Result() (UpdateConfigurationResponse, error) (line 4382)
- (ConfigurationManager_updateConfiguration_Results) HasResult() bool (line 4387)
- (ConfigurationManager_updateConfiguration_Results) SetResult(v UpdateConfigurationResponse) error (line 4392)
- (ConfigurationManager_updateConfiguration_Results) NewResult() (UpdateConfigurationResponse, error) (line 4398)
- NewConfigurationManager_updateConfiguration_Results_List(s *capnp.Segment, sz int32) (ConfigurationManager_updateConfiguration_Results_List, error) (line 4411)
- (ConfigurationManager_updateConfiguration_Results_List) At(i int) ConfigurationManager_updateConfiguration_Results (line 4416)
- (ConfigurationManager_updateConfiguration_Results_List) Set(i int, v ConfigurationManager_updateConfiguration_Results) error (line 4420)
- (ConfigurationManager_updateConfiguration_Results_List) String() string (line 4424)
- (ConfigurationManager_updateConfiguration_Results_Promise) Struct() (ConfigurationManager_updateConfiguration_Results, error) (line 4432)
- (ConfigurationManager_updateConfiguration_Results_Promise) Result() UpdateConfigurationResponse_Promise (line 4437)
- (CloudflaredServer) RegisterUdpSession(ctx context.Context, params func(SessionManager_registerUdpSession_Params) error, opts ...capnp.CallOption) SessionManager_registerUdpSession_Results_Promise (line 4446)
- (CloudflaredServer) UnregisterUdpSession(ctx context.Context, params func(SessionManager_unregisterUdpSession_Params) error, opts ...capnp.CallOption) SessionManager_unregisterUdpSession_Results_Promise (line 4466)
- (CloudflaredServer) UpdateConfiguration(ctx context.Context, params func(ConfigurationManager_updateConfiguration_Params) error, opts ...capnp.CallOption) ConfigurationManager_updateConfiguration_Results_Promise (line 4486)
- CloudflaredServer_ServerToClient(s CloudflaredServer_Server) CloudflaredServer (line 4515)
- CloudflaredServer_Methods(methods []server.Method, s CloudflaredServer_Server) []server.Method (line 4520)
- init() (line 4797)

## Internal Function Surface

- None detected.

## Input Contract

- func-param:c string
- func-param:ctx context.Context
- func-param:i int
- func-param:methods []server.Method
- func-param:msg *capnp.Message
- func-param:n int32
- func-param:opts ...capnp.CallOption
- func-param:params func(ConfigurationManager_updateConfiguration_Params) error
- func-param:params func(RegistrationServer_registerConnection_Params) error
- func-param:params func(RegistrationServer_unregisterConnection_Params) error
- func-param:params func(RegistrationServer_updateLocalConfiguration_Params) error
- func-param:params func(SessionManager_registerUdpSession_Params) error
- func-param:params func(SessionManager_unregisterUdpSession_Params) error
- func-param:params func(TunnelServer_authenticate_Params) error
- func-param:params func(TunnelServer_getServerInfo_Params) error
- func-param:params func(TunnelServer_obsoleteDeclarativeTunnelConnect_Params) error
- func-param:params func(TunnelServer_reconnectTunnel_Params) error
- func-param:params func(TunnelServer_registerTunnel_Params) error
- func-param:params func(TunnelServer_unregisterTunnel_Params) error
- func-param:s *capnp.Segment
- func-param:s CloudflaredServer_Server
- func-param:s ConfigurationManager_Server
- func-param:s RegistrationServer_Server
- func-param:s SessionManager_Server
- func-param:s TunnelServer_Server
- func-param:sz int32
- func-param:v AuthenticateResponse
- func-param:v Authentication
- func-param:v ClientInfo
- func-param:v ConfigurationManager_updateConfiguration_Params
- func-param:v ConfigurationManager_updateConfiguration_Results
- func-param:v ConnectionDetails
- func-param:v ConnectionError
- func-param:v ConnectionOptions
- func-param:v ConnectionResponse
- func-param:v ExistingTunnelPolicy
- func-param:v RegisterUdpSessionResponse
- func-param:v RegistrationOptions
- func-param:v RegistrationServer_registerConnection_Params
- func-param:v RegistrationServer_registerConnection_Results
- func-param:v RegistrationServer_unregisterConnection_Params
- func-param:v RegistrationServer_unregisterConnection_Results
- func-param:v RegistrationServer_updateLocalConfiguration_Params
- func-param:v RegistrationServer_updateLocalConfiguration_Results
- func-param:v ServerInfo
- func-param:v SessionManager_registerUdpSession_Params
- func-param:v SessionManager_registerUdpSession_Results
- func-param:v SessionManager_unregisterUdpSession_Params
- func-param:v SessionManager_unregisterUdpSession_Results
- func-param:v Tag
- func-param:v Tag_List
- func-param:v TunnelAuth
- func-param:v TunnelRegistration
- func-param:v TunnelServer_authenticate_Params
- func-param:v TunnelServer_authenticate_Results
- func-param:v TunnelServer_getServerInfo_Params
- func-param:v TunnelServer_getServerInfo_Results
- func-param:v TunnelServer_obsoleteDeclarativeTunnelConnect_Params
- func-param:v TunnelServer_obsoleteDeclarativeTunnelConnect_Results
- func-param:v TunnelServer_reconnectTunnel_Params
- func-param:v TunnelServer_reconnectTunnel_Results
- func-param:v TunnelServer_registerTunnel_Params
- func-param:v TunnelServer_registerTunnel_Results
- func-param:v TunnelServer_unregisterTunnel_Params
- func-param:v TunnelServer_unregisterTunnel_Results
- func-param:v UpdateConfigurationResponse
- func-param:v []byte
- func-param:v bool
- func-param:v capnp.TextList
- func-param:v int32
- func-param:v int64
- func-param:v string
- func-param:v uint16
- func-param:v uint64
- func-param:v uint8

## Output Contract

- return:AuthenticateResponse
- return:AuthenticateResponse_List
- return:AuthenticateResponse_Promise
- return:Authentication
- return:Authentication_List
- return:ClientInfo
- return:ClientInfo_List
- return:ClientInfo_Promise
- return:CloudflaredServer
- return:ConfigurationManager
- return:ConfigurationManager_updateConfiguration_Params
- return:ConfigurationManager_updateConfiguration_Params_List
- return:ConfigurationManager_updateConfiguration_Results
- return:ConfigurationManager_updateConfiguration_Results_List
- return:ConfigurationManager_updateConfiguration_Results_Promise
- return:ConnectionDetails
- return:ConnectionDetails_List
- return:ConnectionDetails_Promise
- return:ConnectionError
- return:ConnectionError_List
- return:ConnectionError_Promise
- return:ConnectionOptions
- return:ConnectionOptions_List
- return:ConnectionOptions_Promise
- return:ConnectionResponse
- return:ConnectionResponse_List
- return:ConnectionResponse_Promise
- return:ConnectionResponse_result
- return:ConnectionResponse_result_Promise
- return:ConnectionResponse_result_Which
- return:ExistingTunnelPolicy
- return:ExistingTunnelPolicy_List
- return:RegisterUdpSessionResponse
- return:RegisterUdpSessionResponse_List
- return:RegisterUdpSessionResponse_Promise
- return:RegistrationOptions
- return:RegistrationOptions_List
- return:RegistrationOptions_Promise
- return:RegistrationServer
- return:RegistrationServer_registerConnection_Params
- return:RegistrationServer_registerConnection_Params_List
- return:RegistrationServer_registerConnection_Results
- return:RegistrationServer_registerConnection_Results_List
- return:RegistrationServer_registerConnection_Results_Promise
- return:RegistrationServer_unregisterConnection_Params
- return:RegistrationServer_unregisterConnection_Params_List
- return:RegistrationServer_unregisterConnection_Results
- return:RegistrationServer_unregisterConnection_Results_List
- return:RegistrationServer_unregisterConnection_Results_Promise
- return:RegistrationServer_updateLocalConfiguration_Params
- return:RegistrationServer_updateLocalConfiguration_Params_List
- return:RegistrationServer_updateLocalConfiguration_Results
- return:RegistrationServer_updateLocalConfiguration_Results_List
- return:RegistrationServer_updateLocalConfiguration_Results_Promise
- return:ServerInfo
- return:ServerInfo_List
- return:ServerInfo_Promise
- return:SessionManager
- return:SessionManager_registerUdpSession_Params
- return:SessionManager_registerUdpSession_Params_List
- return:SessionManager_registerUdpSession_Results
- return:SessionManager_registerUdpSession_Results_List
- return:SessionManager_registerUdpSession_Results_Promise
- return:SessionManager_unregisterUdpSession_Params
- return:SessionManager_unregisterUdpSession_Params_List
- return:SessionManager_unregisterUdpSession_Results
- return:SessionManager_unregisterUdpSession_Results_List
- return:SessionManager_unregisterUdpSession_Results_Promise
- return:Tag
- return:Tag_List
- return:TunnelAuth
- return:TunnelAuth_List
- return:TunnelAuth_Promise
- return:TunnelRegistration
- return:TunnelRegistration_List
- return:TunnelRegistration_Promise
- return:TunnelServer
- return:TunnelServer_authenticate_Params
- return:TunnelServer_authenticate_Params_List
- return:TunnelServer_authenticate_Results
- return:TunnelServer_authenticate_Results_List
- return:TunnelServer_authenticate_Results_Promise
- return:TunnelServer_getServerInfo_Params
- return:TunnelServer_getServerInfo_Params_List
- return:TunnelServer_getServerInfo_Results
- return:TunnelServer_getServerInfo_Results_List
- return:TunnelServer_getServerInfo_Results_Promise
- return:TunnelServer_obsoleteDeclarativeTunnelConnect_Params
- return:TunnelServer_obsoleteDeclarativeTunnelConnect_Params_List
- return:TunnelServer_obsoleteDeclarativeTunnelConnect_Results
- return:TunnelServer_obsoleteDeclarativeTunnelConnect_Results_List
- return:TunnelServer_obsoleteDeclarativeTunnelConnect_Results_Promise
- return:TunnelServer_reconnectTunnel_Params
- return:TunnelServer_reconnectTunnel_Params_List
- return:TunnelServer_reconnectTunnel_Results
- return:TunnelServer_reconnectTunnel_Results_List
- return:TunnelServer_reconnectTunnel_Results_Promise
- return:TunnelServer_registerTunnel_Params
- return:TunnelServer_registerTunnel_Params_List
- return:TunnelServer_registerTunnel_Results
- return:TunnelServer_registerTunnel_Results_List
- return:TunnelServer_registerTunnel_Results_Promise
- return:TunnelServer_unregisterTunnel_Params
- return:TunnelServer_unregisterTunnel_Params_List
- return:TunnelServer_unregisterTunnel_Results
- return:TunnelServer_unregisterTunnel_Results_List
- return:TunnelServer_unregisterTunnel_Results_Promise
- return:UpdateConfigurationResponse
- return:UpdateConfigurationResponse_List
- return:UpdateConfigurationResponse_Promise
- return:[]byte
- return:[]server.Method
- return:bool
- return:capnp.TextList
- return:error
- return:int32
- return:int64
- return:string
- return:uint16
- return:uint64
- return:uint8

## Side Effects and State Transitions

- network I/O

## Branching and Failure Semantics

- Branch density: if=64, switch=3, select=0
- error-return paths
- panic paths
- fallback/default branches

## Import and Dependency Surface

- golang.org/x/net/context
- strconv
- zombiezen.com/go/capnproto2
- zombiezen.com/go/capnproto2/encoding/text
- zombiezen.com/go/capnproto2/schemas
- zombiezen.com/go/capnproto2/server

## Go-Impl Flow (Intra-file)

```mermaid
flowchart TD
    F1["NewAuthentication"]
    F2["NewRootAuthentication"]
    F3["ReadRootAuthentication"]
    F4["Authentication.String"]
    F5["Authentication.Key"]
    F6["Authentication.HasKey"]
    F7["Authentication.KeyBytes"]
    F8["Authentication.SetKey"]
    F9["Authentication.Email"]
    F10["Authentication.HasEmail"]
    F11["Authentication.EmailBytes"]
    F12["Authentication.SetEmail"]
    F13["Authentication.OriginCAKey"]
    F14["Authentication.HasOriginCAKey"]
    F15["Authentication.OriginCAKeyBytes"]
    F16["Authentication.SetOriginCAKey"]
    F17["NewAuthentication_List"]
    F18["Authentication_List.At"]
    F19["Authentication_List.Set"]
    F20["Authentication_List.String"]
    F21["Authentication_Promise.Struct"]
    F22["NewTunnelRegistration"]
    F23["NewRootTunnelRegistration"]
    F24["ReadRootTunnelRegistration"]
    F25["TunnelRegistration.String"]
    F26["TunnelRegistration.Err"]
    F27["TunnelRegistration.HasErr"]
    F28["TunnelRegistration.ErrBytes"]
    F29["TunnelRegistration.SetErr"]
    F30["TunnelRegistration.Url"]
    F31["TunnelRegistration.HasUrl"]
    F32["TunnelRegistration.UrlBytes"]
    F33["TunnelRegistration.SetUrl"]
    F34["TunnelRegistration.LogLines"]
    F35["TunnelRegistration.HasLogLines"]
    F36["TunnelRegistration.SetLogLines"]
    F37["TunnelRegistration.NewLogLines"]
    F38["TunnelRegistration.PermanentFailure"]
    F39["TunnelRegistration.SetPermanentFailure"]
    F40["TunnelRegistration.TunnelID"]
    F41["TunnelRegistration.HasTunnelID"]
    F42["TunnelRegistration.TunnelIDBytes"]
    F43["TunnelRegistration.SetTunnelID"]
    F44["TunnelRegistration.RetryAfterSeconds"]
    F45["TunnelRegistration.SetRetryAfterSeconds"]
    F46["TunnelRegistration.EventDigest"]
    F47["TunnelRegistration.HasEventDigest"]
    F48["TunnelRegistration.SetEventDigest"]
    F49["TunnelRegistration.ConnDigest"]
    F50["TunnelRegistration.HasConnDigest"]
    F51["TunnelRegistration.SetConnDigest"]
    F52["NewTunnelRegistration_List"]
    F53["TunnelRegistration_List.At"]
    F54["TunnelRegistration_List.Set"]
    F55["TunnelRegistration_List.String"]
    F56["TunnelRegistration_Promise.Struct"]
    F57["NewRegistrationOptions"]
    F58["NewRootRegistrationOptions"]
    F59["ReadRootRegistrationOptions"]
    F60["RegistrationOptions.String"]
    F61["RegistrationOptions.ClientId"]
    F62["RegistrationOptions.HasClientId"]
    F63["RegistrationOptions.ClientIdBytes"]
    F64["RegistrationOptions.SetClientId"]
    F65["RegistrationOptions.Version"]
    F66["RegistrationOptions.HasVersion"]
    F67["RegistrationOptions.VersionBytes"]
    F68["RegistrationOptions.SetVersion"]
    F69["RegistrationOptions.Os"]
    F70["RegistrationOptions.HasOs"]
    F71["RegistrationOptions.OsBytes"]
    F72["RegistrationOptions.SetOs"]
    F73["RegistrationOptions.ExistingTunnelPolicy"]
    F74["RegistrationOptions.SetExistingTunnelPolicy"]
    F75["RegistrationOptions.PoolName"]
    F76["RegistrationOptions.HasPoolName"]
    F77["RegistrationOptions.PoolNameBytes"]
    F78["RegistrationOptions.SetPoolName"]
    F79["RegistrationOptions.Tags"]
    F80["RegistrationOptions.HasTags"]
    F81["RegistrationOptions.SetTags"]
    F82["RegistrationOptions.NewTags"]
    F83["RegistrationOptions.ConnectionId"]
    F84["RegistrationOptions.SetConnectionId"]
    F85["RegistrationOptions.OriginLocalIp"]
    F86["RegistrationOptions.HasOriginLocalIp"]
    F87["RegistrationOptions.OriginLocalIpBytes"]
    F88["RegistrationOptions.SetOriginLocalIp"]
    F89["RegistrationOptions.IsAutoupdated"]
    F90["RegistrationOptions.SetIsAutoupdated"]
    F91["RegistrationOptions.RunFromTerminal"]
    F92["RegistrationOptions.SetRunFromTerminal"]
    F93["RegistrationOptions.CompressionQuality"]
    F94["RegistrationOptions.SetCompressionQuality"]
    F95["RegistrationOptions.Uuid"]
    F96["RegistrationOptions.HasUuid"]
    F97["RegistrationOptions.UuidBytes"]
    F98["RegistrationOptions.SetUuid"]
    F99["RegistrationOptions.NumPreviousAttempts"]
    F100["RegistrationOptions.SetNumPreviousAttempts"]
    F101["RegistrationOptions.Features"]
    F102["RegistrationOptions.HasFeatures"]
    F103["RegistrationOptions.SetFeatures"]
    F104["RegistrationOptions.NewFeatures"]
    F105["NewRegistrationOptions_List"]
    F106["RegistrationOptions_List.At"]
    F107["RegistrationOptions_List.Set"]
    F108["RegistrationOptions_List.String"]
    F109["RegistrationOptions_Promise.Struct"]
    F110["ExistingTunnelPolicy.String"]
    F111["ExistingTunnelPolicyFromString"]
    F112["NewExistingTunnelPolicy_List"]
    F113["ExistingTunnelPolicy_List.At"]
    F114["ExistingTunnelPolicy_List.Set"]
    F115["NewServerInfo"]
    F116["NewRootServerInfo"]
    F117["ReadRootServerInfo"]
    F118["ServerInfo.String"]
    F119["ServerInfo.LocationName"]
    F120["ServerInfo.HasLocationName"]
    F121["ServerInfo.LocationNameBytes"]
    F122["ServerInfo.SetLocationName"]
    F123["NewServerInfo_List"]
    F124["ServerInfo_List.At"]
    F125["ServerInfo_List.Set"]
    F126["ServerInfo_List.String"]
    F127["ServerInfo_Promise.Struct"]
    F128["NewAuthenticateResponse"]
    F129["NewRootAuthenticateResponse"]
    F130["ReadRootAuthenticateResponse"]
    F131["AuthenticateResponse.String"]
    F132["AuthenticateResponse.PermanentErr"]
    F133["AuthenticateResponse.HasPermanentErr"]
    F134["AuthenticateResponse.PermanentErrBytes"]
    F135["AuthenticateResponse.SetPermanentErr"]
    F136["AuthenticateResponse.RetryableErr"]
    F137["AuthenticateResponse.HasRetryableErr"]
    F138["AuthenticateResponse.RetryableErrBytes"]
    F139["AuthenticateResponse.SetRetryableErr"]
    F140["AuthenticateResponse.Jwt"]
    F141["AuthenticateResponse.HasJwt"]
    F142["AuthenticateResponse.SetJwt"]
    F143["AuthenticateResponse.HoursUntilRefresh"]
    F144["AuthenticateResponse.SetHoursUntilRefresh"]
    F145["NewAuthenticateResponse_List"]
    F146["AuthenticateResponse_List.At"]
    F147["AuthenticateResponse_List.Set"]
    F148["AuthenticateResponse_List.String"]
    F149["AuthenticateResponse_Promise.Struct"]
    F150["TunnelServer.RegisterTunnel"]
    F151["TunnelServer.GetServerInfo"]
    F152["TunnelServer.UnregisterTunnel"]
    F153["TunnelServer.ObsoleteDeclarativeTunnelConnect"]
    F154["TunnelServer.Authenticate"]
    F155["TunnelServer.ReconnectTunnel"]
    F156["TunnelServer.RegisterConnection"]
    F157["TunnelServer.UnregisterConnection"]
    F158["TunnelServer.UpdateLocalConfiguration"]
    F159["TunnelServer_ServerToClient"]
    F160["TunnelServer_Methods"]
    F161["NewTunnelServer_registerTunnel_Params"]
    F162["NewRootTunnelServer_registerTunnel_Params"]
    F163["ReadRootTunnelServer_registerTunnel_Params"]
    F164["TunnelServer_registerTunnel_Params.String"]
    F165["TunnelServer_registerTunnel_Params.OriginCert"]
    F166["TunnelServer_registerTunnel_Params.HasOriginCert"]
    F167["TunnelServer_registerTunnel_Params.SetOriginCert"]
    F168["TunnelServer_registerTunnel_Params.Hostname"]
    F169["TunnelServer_registerTunnel_Params.HasHostname"]
    F170["TunnelServer_registerTunnel_Params.HostnameBytes"]
    F171["TunnelServer_registerTunnel_Params.SetHostname"]
    F172["TunnelServer_registerTunnel_Params.Options"]
    F173["TunnelServer_registerTunnel_Params.HasOptions"]
    F174["TunnelServer_registerTunnel_Params.SetOptions"]
    F175["TunnelServer_registerTunnel_Params.NewOptions"]
    F176["NewTunnelServer_registerTunnel_Params_List"]
    F177["TunnelServer_registerTunnel_Params_List.At"]
    F178["TunnelServer_registerTunnel_Params_List.Set"]
    F179["TunnelServer_registerTunnel_Params_List.String"]
    F180["TunnelServer_registerTunnel_Params_Promise.Struct"]
    F181["TunnelServer_registerTunnel_Params_Promise.Options"]
    F182["NewTunnelServer_registerTunnel_Results"]
    F183["NewRootTunnelServer_registerTunnel_Results"]
    F184["ReadRootTunnelServer_registerTunnel_Results"]
    F185["TunnelServer_registerTunnel_Results.String"]
    F186["TunnelServer_registerTunnel_Results.Result"]
    F187["TunnelServer_registerTunnel_Results.HasResult"]
    F188["TunnelServer_registerTunnel_Results.SetResult"]
    F189["TunnelServer_registerTunnel_Results.NewResult"]
    F190["NewTunnelServer_registerTunnel_Results_List"]
    F191["TunnelServer_registerTunnel_Results_List.At"]
    F192["TunnelServer_registerTunnel_Results_List.Set"]
    F193["TunnelServer_registerTunnel_Results_List.String"]
    F194["TunnelServer_registerTunnel_Results_Promise.Struct"]
    F195["TunnelServer_registerTunnel_Results_Promise.Result"]
    F196["NewTunnelServer_getServerInfo_Params"]
    F197["NewRootTunnelServer_getServerInfo_Params"]
    F198["ReadRootTunnelServer_getServerInfo_Params"]
    F199["TunnelServer_getServerInfo_Params.String"]
    F200["NewTunnelServer_getServerInfo_Params_List"]
    F201["TunnelServer_getServerInfo_Params_List.At"]
    F202["TunnelServer_getServerInfo_Params_List.Set"]
    F203["TunnelServer_getServerInfo_Params_List.String"]
    F204["TunnelServer_getServerInfo_Params_Promise.Struct"]
    F205["NewTunnelServer_getServerInfo_Results"]
    F206["NewRootTunnelServer_getServerInfo_Results"]
    F207["ReadRootTunnelServer_getServerInfo_Results"]
    F208["TunnelServer_getServerInfo_Results.String"]
    F209["TunnelServer_getServerInfo_Results.Result"]
    F210["TunnelServer_getServerInfo_Results.HasResult"]
    F211["TunnelServer_getServerInfo_Results.SetResult"]
    F212["TunnelServer_getServerInfo_Results.NewResult"]
    F213["NewTunnelServer_getServerInfo_Results_List"]
    F214["TunnelServer_getServerInfo_Results_List.At"]
    F215["TunnelServer_getServerInfo_Results_List.Set"]
    F216["TunnelServer_getServerInfo_Results_List.String"]
    F217["TunnelServer_getServerInfo_Results_Promise.Struct"]
    F218["TunnelServer_getServerInfo_Results_Promise.Result"]
    F219["NewTunnelServer_unregisterTunnel_Params"]
    F220["NewRootTunnelServer_unregisterTunnel_Params"]
    F221["ReadRootTunnelServer_unregisterTunnel_Params"]
    F222["TunnelServer_unregisterTunnel_Params.String"]
    F223["TunnelServer_unregisterTunnel_Params.GracePeriodNanoSec"]
    F224["TunnelServer_unregisterTunnel_Params.SetGracePeriodNanoSec"]
    F225["NewTunnelServer_unregisterTunnel_Params_List"]
    F226["TunnelServer_unregisterTunnel_Params_List.At"]
    F227["TunnelServer_unregisterTunnel_Params_List.Set"]
    F228["TunnelServer_unregisterTunnel_Params_List.String"]
    F229["TunnelServer_unregisterTunnel_Params_Promise.Struct"]
    F230["NewTunnelServer_unregisterTunnel_Results"]
    F231["NewRootTunnelServer_unregisterTunnel_Results"]
    F232["ReadRootTunnelServer_unregisterTunnel_Results"]
    F233["TunnelServer_unregisterTunnel_Results.String"]
    F234["NewTunnelServer_unregisterTunnel_Results_List"]
    F235["TunnelServer_unregisterTunnel_Results_List.At"]
    F236["TunnelServer_unregisterTunnel_Results_List.Set"]
    F237["TunnelServer_unregisterTunnel_Results_List.String"]
    F238["TunnelServer_unregisterTunnel_Results_Promise.Struct"]
    F239["NewTunnelServer_obsoleteDeclarativeTunnelConnect_Params"]
    F240["NewRootTunnelServer_obsoleteDeclarativeTunnelConnect_Params"]
    F241["ReadRootTunnelServer_obsoleteDeclarativeTunnelConnect_Params"]
    F242["TunnelServer_obsoleteDeclarativeTunnelConnect_Params.String"]
    F243["NewTunnelServer_obsoleteDeclarativeTunnelConnect_Params_List"]
    F244["TunnelServer_obsoleteDeclarativeTunnelConnect_Params_List.At"]
    F245["TunnelServer_obsoleteDeclarativeTunnelConnect_Params_List.Set"]
    F246["TunnelServer_obsoleteDeclarativeTunnelConnect_Params_List.String"]
    F247["TunnelServer_obsoleteDeclarativeTunnelConnect_Params_Promise.Struct"]
    F248["NewTunnelServer_obsoleteDeclarativeTunnelConnect_Results"]
    F249["NewRootTunnelServer_obsoleteDeclarativeTunnelConnect_Results"]
    F250["ReadRootTunnelServer_obsoleteDeclarativeTunnelConnect_Results"]
    F251["TunnelServer_obsoleteDeclarativeTunnelConnect_Results.String"]
    F252["NewTunnelServer_obsoleteDeclarativeTunnelConnect_Results_List"]
    F253["TunnelServer_obsoleteDeclarativeTunnelConnect_Results_List.At"]
    F254["TunnelServer_obsoleteDeclarativeTunnelConnect_Results_List.Set"]
    F255["TunnelServer_obsoleteDeclarativeTunnelConnect_Results_List.String"]
    F256["TunnelServer_obsoleteDeclarativeTunnelConnect_Results_Promise.Struct"]
    F257["NewTunnelServer_authenticate_Params"]
    F258["NewRootTunnelServer_authenticate_Params"]
    F259["ReadRootTunnelServer_authenticate_Params"]
    F260["TunnelServer_authenticate_Params.String"]
    F261["TunnelServer_authenticate_Params.OriginCert"]
    F262["TunnelServer_authenticate_Params.HasOriginCert"]
    F263["TunnelServer_authenticate_Params.SetOriginCert"]
    F264["TunnelServer_authenticate_Params.Hostname"]
    F265["TunnelServer_authenticate_Params.HasHostname"]
    F266["TunnelServer_authenticate_Params.HostnameBytes"]
    F267["TunnelServer_authenticate_Params.SetHostname"]
    F268["TunnelServer_authenticate_Params.Options"]
    F269["TunnelServer_authenticate_Params.HasOptions"]
    F270["TunnelServer_authenticate_Params.SetOptions"]
    F271["TunnelServer_authenticate_Params.NewOptions"]
    F272["NewTunnelServer_authenticate_Params_List"]
    F273["TunnelServer_authenticate_Params_List.At"]
    F274["TunnelServer_authenticate_Params_List.Set"]
    F275["TunnelServer_authenticate_Params_List.String"]
    F276["TunnelServer_authenticate_Params_Promise.Struct"]
    F277["TunnelServer_authenticate_Params_Promise.Options"]
    F278["NewTunnelServer_authenticate_Results"]
    F279["NewRootTunnelServer_authenticate_Results"]
    F280["ReadRootTunnelServer_authenticate_Results"]
    F281["TunnelServer_authenticate_Results.String"]
    F282["TunnelServer_authenticate_Results.Result"]
    F283["TunnelServer_authenticate_Results.HasResult"]
    F284["TunnelServer_authenticate_Results.SetResult"]
    F285["TunnelServer_authenticate_Results.NewResult"]
    F286["NewTunnelServer_authenticate_Results_List"]
    F287["TunnelServer_authenticate_Results_List.At"]
    F288["TunnelServer_authenticate_Results_List.Set"]
    F289["TunnelServer_authenticate_Results_List.String"]
    F290["TunnelServer_authenticate_Results_Promise.Struct"]
    F291["TunnelServer_authenticate_Results_Promise.Result"]
    F292["NewTunnelServer_reconnectTunnel_Params"]
    F293["NewRootTunnelServer_reconnectTunnel_Params"]
    F294["ReadRootTunnelServer_reconnectTunnel_Params"]
    F295["TunnelServer_reconnectTunnel_Params.String"]
    F296["TunnelServer_reconnectTunnel_Params.Jwt"]
    F297["TunnelServer_reconnectTunnel_Params.HasJwt"]
    F298["TunnelServer_reconnectTunnel_Params.SetJwt"]
    F299["TunnelServer_reconnectTunnel_Params.EventDigest"]
    F300["TunnelServer_reconnectTunnel_Params.HasEventDigest"]
    F301["TunnelServer_reconnectTunnel_Params.SetEventDigest"]
    F302["TunnelServer_reconnectTunnel_Params.ConnDigest"]
    F303["TunnelServer_reconnectTunnel_Params.HasConnDigest"]
    F304["TunnelServer_reconnectTunnel_Params.SetConnDigest"]
    F305["TunnelServer_reconnectTunnel_Params.Hostname"]
    F306["TunnelServer_reconnectTunnel_Params.HasHostname"]
    F307["TunnelServer_reconnectTunnel_Params.HostnameBytes"]
    F308["TunnelServer_reconnectTunnel_Params.SetHostname"]
    F309["TunnelServer_reconnectTunnel_Params.Options"]
    F310["TunnelServer_reconnectTunnel_Params.HasOptions"]
    F311["TunnelServer_reconnectTunnel_Params.SetOptions"]
    F312["TunnelServer_reconnectTunnel_Params.NewOptions"]
    F313["NewTunnelServer_reconnectTunnel_Params_List"]
    F314["TunnelServer_reconnectTunnel_Params_List.At"]
    F315["TunnelServer_reconnectTunnel_Params_List.Set"]
    F316["TunnelServer_reconnectTunnel_Params_List.String"]
    F317["TunnelServer_reconnectTunnel_Params_Promise.Struct"]
    F318["TunnelServer_reconnectTunnel_Params_Promise.Options"]
    F319["NewTunnelServer_reconnectTunnel_Results"]
    F320["NewRootTunnelServer_reconnectTunnel_Results"]
    F321["ReadRootTunnelServer_reconnectTunnel_Results"]
    F322["TunnelServer_reconnectTunnel_Results.String"]
    F323["TunnelServer_reconnectTunnel_Results.Result"]
    F324["TunnelServer_reconnectTunnel_Results.HasResult"]
    F325["TunnelServer_reconnectTunnel_Results.SetResult"]
    F326["TunnelServer_reconnectTunnel_Results.NewResult"]
    F327["NewTunnelServer_reconnectTunnel_Results_List"]
    F328["TunnelServer_reconnectTunnel_Results_List.At"]
    F329["TunnelServer_reconnectTunnel_Results_List.Set"]
    F330["TunnelServer_reconnectTunnel_Results_List.String"]
    F331["TunnelServer_reconnectTunnel_Results_Promise.Struct"]
    F332["TunnelServer_reconnectTunnel_Results_Promise.Result"]
    F333["NewTag"]
    F334["NewRootTag"]
    F335["ReadRootTag"]
    F336["Tag.String"]
    F337["Tag.Name"]
    F338["Tag.HasName"]
    F339["Tag.NameBytes"]
    F340["Tag.SetName"]
    F341["Tag.Value"]
    F342["Tag.HasValue"]
    F343["Tag.ValueBytes"]
    F344["Tag.SetValue"]
    F345["NewTag_List"]
    F346["Tag_List.At"]
    F347["Tag_List.Set"]
    F348["Tag_List.String"]
    F349["Tag_Promise.Struct"]
    F350["NewClientInfo"]
    F351["NewRootClientInfo"]
    F352["ReadRootClientInfo"]
    F353["ClientInfo.String"]
    F354["ClientInfo.ClientId"]
    F355["ClientInfo.HasClientId"]
    F356["ClientInfo.SetClientId"]
    F357["ClientInfo.Features"]
    F358["ClientInfo.HasFeatures"]
    F359["ClientInfo.SetFeatures"]
    F360["ClientInfo.NewFeatures"]
    F361["ClientInfo.Version"]
    F362["ClientInfo.HasVersion"]
    F363["ClientInfo.VersionBytes"]
    F364["ClientInfo.SetVersion"]
    F365["ClientInfo.Arch"]
    F366["ClientInfo.HasArch"]
    F367["ClientInfo.ArchBytes"]
    F368["ClientInfo.SetArch"]
    F369["NewClientInfo_List"]
    F370["ClientInfo_List.At"]
    F371["ClientInfo_List.Set"]
    F372["ClientInfo_List.String"]
    F373["ClientInfo_Promise.Struct"]
    F374["NewConnectionOptions"]
    F375["NewRootConnectionOptions"]
    F376["ReadRootConnectionOptions"]
    F377["ConnectionOptions.String"]
    F378["ConnectionOptions.Client"]
    F379["ConnectionOptions.HasClient"]
    F380["ConnectionOptions.SetClient"]
    F381["ConnectionOptions.NewClient"]
    F382["ConnectionOptions.OriginLocalIp"]
    F383["ConnectionOptions.HasOriginLocalIp"]
    F384["ConnectionOptions.SetOriginLocalIp"]
    F385["ConnectionOptions.ReplaceExisting"]
    F386["ConnectionOptions.SetReplaceExisting"]
    F387["ConnectionOptions.CompressionQuality"]
    F388["ConnectionOptions.SetCompressionQuality"]
    F389["ConnectionOptions.NumPreviousAttempts"]
    F390["ConnectionOptions.SetNumPreviousAttempts"]
    F391["NewConnectionOptions_List"]
    F392["ConnectionOptions_List.At"]
    F393["ConnectionOptions_List.Set"]
    F394["ConnectionOptions_List.String"]
    F395["ConnectionOptions_Promise.Struct"]
    F396["ConnectionOptions_Promise.Client"]
    F397["ConnectionResponse_result_Which.String"]
    F398["NewConnectionResponse"]
    F399["NewRootConnectionResponse"]
    F400["ReadRootConnectionResponse"]
    F401["ConnectionResponse.String"]
    F402["ConnectionResponse.Result"]
    F403["ConnectionResponse_result.Which"]
    F404["ConnectionResponse_result.Error"]
    F405["ConnectionResponse_result.HasError"]
    F406["ConnectionResponse_result.SetError"]
    F407["ConnectionResponse_result.NewError"]
    F408["ConnectionResponse_result.ConnectionDetails"]
    F409["ConnectionResponse_result.HasConnectionDetails"]
    F410["ConnectionResponse_result.SetConnectionDetails"]
    F411["ConnectionResponse_result.NewConnectionDetails"]
    F412["NewConnectionResponse_List"]
    F413["ConnectionResponse_List.At"]
    F414["ConnectionResponse_List.Set"]
    F415["ConnectionResponse_List.String"]
    F416["ConnectionResponse_Promise.Struct"]
    F417["ConnectionResponse_Promise.Result"]
    F418["ConnectionResponse_result_Promise.Struct"]
    F419["ConnectionResponse_result_Promise.Error"]
    F420["ConnectionResponse_result_Promise.ConnectionDetails"]
    F421["NewConnectionError"]
    F422["NewRootConnectionError"]
    F423["ReadRootConnectionError"]
    F424["ConnectionError.String"]
    F425["ConnectionError.Cause"]
    F426["ConnectionError.HasCause"]
    F427["ConnectionError.CauseBytes"]
    F428["ConnectionError.SetCause"]
    F429["ConnectionError.RetryAfter"]
    F430["ConnectionError.SetRetryAfter"]
    F431["ConnectionError.ShouldRetry"]
    F432["ConnectionError.SetShouldRetry"]
    F433["NewConnectionError_List"]
    F434["ConnectionError_List.At"]
    F435["ConnectionError_List.Set"]
    F436["ConnectionError_List.String"]
    F437["ConnectionError_Promise.Struct"]
    F438["NewConnectionDetails"]
    F439["NewRootConnectionDetails"]
    F440["ReadRootConnectionDetails"]
    F441["ConnectionDetails.String"]
    F442["ConnectionDetails.Uuid"]
    F443["ConnectionDetails.HasUuid"]
    F444["ConnectionDetails.SetUuid"]
    F445["ConnectionDetails.LocationName"]
    F446["ConnectionDetails.HasLocationName"]
    F447["ConnectionDetails.LocationNameBytes"]
    F448["ConnectionDetails.SetLocationName"]
    F449["ConnectionDetails.TunnelIsRemotelyManaged"]
    F450["ConnectionDetails.SetTunnelIsRemotelyManaged"]
    F451["NewConnectionDetails_List"]
    F452["ConnectionDetails_List.At"]
    F453["ConnectionDetails_List.Set"]
    F454["ConnectionDetails_List.String"]
    F455["ConnectionDetails_Promise.Struct"]
    F456["NewTunnelAuth"]
    F457["NewRootTunnelAuth"]
    F458["ReadRootTunnelAuth"]
    F459["TunnelAuth.String"]
    F460["TunnelAuth.AccountTag"]
    F461["TunnelAuth.HasAccountTag"]
    F462["TunnelAuth.AccountTagBytes"]
    F463["TunnelAuth.SetAccountTag"]
    F464["TunnelAuth.TunnelSecret"]
    F465["TunnelAuth.HasTunnelSecret"]
    F466["TunnelAuth.SetTunnelSecret"]
    F467["NewTunnelAuth_List"]
    F468["TunnelAuth_List.At"]
    F469["TunnelAuth_List.Set"]
    F470["TunnelAuth_List.String"]
    F471["TunnelAuth_Promise.Struct"]
    F472["RegistrationServer.RegisterConnection"]
    F473["RegistrationServer.UnregisterConnection"]
    F474["RegistrationServer.UpdateLocalConfiguration"]
    F475["RegistrationServer_ServerToClient"]
    F476["RegistrationServer_Methods"]
    F477["NewRegistrationServer_registerConnection_Params"]
    F478["NewRootRegistrationServer_registerConnection_Params"]
    F479["ReadRootRegistrationServer_registerConnection_Params"]
    F480["RegistrationServer_registerConnection_Params.String"]
    F481["RegistrationServer_registerConnection_Params.Auth"]
    F482["RegistrationServer_registerConnection_Params.HasAuth"]
    F483["RegistrationServer_registerConnection_Params.SetAuth"]
    F484["RegistrationServer_registerConnection_Params.NewAuth"]
    F485["RegistrationServer_registerConnection_Params.TunnelId"]
    F486["RegistrationServer_registerConnection_Params.HasTunnelId"]
    F487["RegistrationServer_registerConnection_Params.SetTunnelId"]
    F488["RegistrationServer_registerConnection_Params.ConnIndex"]
    F489["RegistrationServer_registerConnection_Params.SetConnIndex"]
    F490["RegistrationServer_registerConnection_Params.Options"]
    F491["RegistrationServer_registerConnection_Params.HasOptions"]
    F492["RegistrationServer_registerConnection_Params.SetOptions"]
    F493["RegistrationServer_registerConnection_Params.NewOptions"]
    F494["NewRegistrationServer_registerConnection_Params_List"]
    F495["RegistrationServer_registerConnection_Params_List.At"]
    F496["RegistrationServer_registerConnection_Params_List.Set"]
    F497["RegistrationServer_registerConnection_Params_List.String"]
    F498["RegistrationServer_registerConnection_Params_Promise.Struct"]
    F499["RegistrationServer_registerConnection_Params_Promise.Auth"]
    F500["RegistrationServer_registerConnection_Params_Promise.Options"]
    F501["NewRegistrationServer_registerConnection_Results"]
    F502["NewRootRegistrationServer_registerConnection_Results"]
    F503["ReadRootRegistrationServer_registerConnection_Results"]
    F504["RegistrationServer_registerConnection_Results.String"]
    F505["RegistrationServer_registerConnection_Results.Result"]
    F506["RegistrationServer_registerConnection_Results.HasResult"]
    F507["RegistrationServer_registerConnection_Results.SetResult"]
    F508["RegistrationServer_registerConnection_Results.NewResult"]
    F509["NewRegistrationServer_registerConnection_Results_List"]
    F510["RegistrationServer_registerConnection_Results_List.At"]
    F511["RegistrationServer_registerConnection_Results_List.Set"]
    F512["RegistrationServer_registerConnection_Results_List.String"]
    F513["RegistrationServer_registerConnection_Results_Promise.Struct"]
    F514["RegistrationServer_registerConnection_Results_Promise.Result"]
    F515["NewRegistrationServer_unregisterConnection_Params"]
    F516["NewRootRegistrationServer_unregisterConnection_Params"]
    F517["ReadRootRegistrationServer_unregisterConnection_Params"]
    F518["RegistrationServer_unregisterConnection_Params.String"]
    F519["NewRegistrationServer_unregisterConnection_Params_List"]
    F520["RegistrationServer_unregisterConnection_Params_List.At"]
    F521["RegistrationServer_unregisterConnection_Params_List.Set"]
    F522["RegistrationServer_unregisterConnection_Params_List.String"]
    F523["RegistrationServer_unregisterConnection_Params_Promise.Struct"]
    F524["NewRegistrationServer_unregisterConnection_Results"]
    F525["NewRootRegistrationServer_unregisterConnection_Results"]
    F526["ReadRootRegistrationServer_unregisterConnection_Results"]
    F527["RegistrationServer_unregisterConnection_Results.String"]
    F528["NewRegistrationServer_unregisterConnection_Results_List"]
    F529["RegistrationServer_unregisterConnection_Results_List.At"]
    F530["RegistrationServer_unregisterConnection_Results_List.Set"]
    F531["RegistrationServer_unregisterConnection_Results_List.String"]
    F532["RegistrationServer_unregisterConnection_Results_Promise.Struct"]
    F533["NewRegistrationServer_updateLocalConfiguration_Params"]
    F534["NewRootRegistrationServer_updateLocalConfiguration_Params"]
    F535["ReadRootRegistrationServer_updateLocalConfiguration_Params"]
    F536["RegistrationServer_updateLocalConfiguration_Params.String"]
    F537["RegistrationServer_updateLocalConfiguration_Params.Config"]
    F538["RegistrationServer_updateLocalConfiguration_Params.HasConfig"]
    F539["RegistrationServer_updateLocalConfiguration_Params.SetConfig"]
    F540["NewRegistrationServer_updateLocalConfiguration_Params_List"]
    F541["RegistrationServer_updateLocalConfiguration_Params_List.At"]
    F542["RegistrationServer_updateLocalConfiguration_Params_List.Set"]
    F543["RegistrationServer_updateLocalConfiguration_Params_List.String"]
    F544["RegistrationServer_updateLocalConfiguration_Params_Promise.Struct"]
    F545["NewRegistrationServer_updateLocalConfiguration_Results"]
    F546["NewRootRegistrationServer_updateLocalConfiguration_Results"]
    F547["ReadRootRegistrationServer_updateLocalConfiguration_Results"]
    F548["RegistrationServer_updateLocalConfiguration_Results.String"]
    F549["NewRegistrationServer_updateLocalConfiguration_Results_List"]
    F550["RegistrationServer_updateLocalConfiguration_Results_List.At"]
    F551["RegistrationServer_updateLocalConfiguration_Results_List.Set"]
    F552["RegistrationServer_updateLocalConfiguration_Results_List.String"]
    F553["RegistrationServer_updateLocalConfiguration_Results_Promise.Struct"]
    F554["NewRegisterUdpSessionResponse"]
    F555["NewRootRegisterUdpSessionResponse"]
    F556["ReadRootRegisterUdpSessionResponse"]
    F557["RegisterUdpSessionResponse.String"]
    F558["RegisterUdpSessionResponse.Err"]
    F559["RegisterUdpSessionResponse.HasErr"]
    F560["RegisterUdpSessionResponse.ErrBytes"]
    F561["RegisterUdpSessionResponse.SetErr"]
    F562["RegisterUdpSessionResponse.Spans"]
    F563["RegisterUdpSessionResponse.HasSpans"]
    F564["RegisterUdpSessionResponse.SetSpans"]
    F565["NewRegisterUdpSessionResponse_List"]
    F566["RegisterUdpSessionResponse_List.At"]
    F567["RegisterUdpSessionResponse_List.Set"]
    F568["RegisterUdpSessionResponse_List.String"]
    F569["RegisterUdpSessionResponse_Promise.Struct"]
    F570["SessionManager.RegisterUdpSession"]
    F571["SessionManager.UnregisterUdpSession"]
    F572["SessionManager_ServerToClient"]
    F573["SessionManager_Methods"]
    F574["NewSessionManager_registerUdpSession_Params"]
    F575["NewRootSessionManager_registerUdpSession_Params"]
    F576["ReadRootSessionManager_registerUdpSession_Params"]
    F577["SessionManager_registerUdpSession_Params.String"]
    F578["SessionManager_registerUdpSession_Params.SessionId"]
    F579["SessionManager_registerUdpSession_Params.HasSessionId"]
    F580["SessionManager_registerUdpSession_Params.SetSessionId"]
    F581["SessionManager_registerUdpSession_Params.DstIp"]
    F582["SessionManager_registerUdpSession_Params.HasDstIp"]
    F583["SessionManager_registerUdpSession_Params.SetDstIp"]
    F584["SessionManager_registerUdpSession_Params.DstPort"]
    F585["SessionManager_registerUdpSession_Params.SetDstPort"]
    F586["SessionManager_registerUdpSession_Params.CloseAfterIdleHint"]
    F587["SessionManager_registerUdpSession_Params.SetCloseAfterIdleHint"]
    F588["SessionManager_registerUdpSession_Params.TraceContext"]
    F589["SessionManager_registerUdpSession_Params.HasTraceContext"]
    F590["SessionManager_registerUdpSession_Params.TraceContextBytes"]
    F591["SessionManager_registerUdpSession_Params.SetTraceContext"]
    F592["NewSessionManager_registerUdpSession_Params_List"]
    F593["SessionManager_registerUdpSession_Params_List.At"]
    F594["SessionManager_registerUdpSession_Params_List.Set"]
    F595["SessionManager_registerUdpSession_Params_List.String"]
    F596["SessionManager_registerUdpSession_Params_Promise.Struct"]
    F597["NewSessionManager_registerUdpSession_Results"]
    F598["NewRootSessionManager_registerUdpSession_Results"]
    F599["ReadRootSessionManager_registerUdpSession_Results"]
    F600["SessionManager_registerUdpSession_Results.String"]
    F601["SessionManager_registerUdpSession_Results.Result"]
    F602["SessionManager_registerUdpSession_Results.HasResult"]
    F603["SessionManager_registerUdpSession_Results.SetResult"]
    F604["SessionManager_registerUdpSession_Results.NewResult"]
    F605["NewSessionManager_registerUdpSession_Results_List"]
    F606["SessionManager_registerUdpSession_Results_List.At"]
    F607["SessionManager_registerUdpSession_Results_List.Set"]
    F608["SessionManager_registerUdpSession_Results_List.String"]
    F609["SessionManager_registerUdpSession_Results_Promise.Struct"]
    F610["SessionManager_registerUdpSession_Results_Promise.Result"]
    F611["NewSessionManager_unregisterUdpSession_Params"]
    F612["NewRootSessionManager_unregisterUdpSession_Params"]
    F613["ReadRootSessionManager_unregisterUdpSession_Params"]
    F614["SessionManager_unregisterUdpSession_Params.String"]
    F615["SessionManager_unregisterUdpSession_Params.SessionId"]
    F616["SessionManager_unregisterUdpSession_Params.HasSessionId"]
    F617["SessionManager_unregisterUdpSession_Params.SetSessionId"]
    F618["SessionManager_unregisterUdpSession_Params.Message"]
    F619["SessionManager_unregisterUdpSession_Params.HasMessage"]
    F620["SessionManager_unregisterUdpSession_Params.MessageBytes"]
    F621["SessionManager_unregisterUdpSession_Params.SetMessage"]
    F622["NewSessionManager_unregisterUdpSession_Params_List"]
    F623["SessionManager_unregisterUdpSession_Params_List.At"]
    F624["SessionManager_unregisterUdpSession_Params_List.Set"]
    F625["SessionManager_unregisterUdpSession_Params_List.String"]
    F626["SessionManager_unregisterUdpSession_Params_Promise.Struct"]
    F627["NewSessionManager_unregisterUdpSession_Results"]
    F628["NewRootSessionManager_unregisterUdpSession_Results"]
    F629["ReadRootSessionManager_unregisterUdpSession_Results"]
    F630["SessionManager_unregisterUdpSession_Results.String"]
    F631["NewSessionManager_unregisterUdpSession_Results_List"]
    F632["SessionManager_unregisterUdpSession_Results_List.At"]
    F633["SessionManager_unregisterUdpSession_Results_List.Set"]
    F634["SessionManager_unregisterUdpSession_Results_List.String"]
    F635["SessionManager_unregisterUdpSession_Results_Promise.Struct"]
    F636["NewUpdateConfigurationResponse"]
    F637["NewRootUpdateConfigurationResponse"]
    F638["ReadRootUpdateConfigurationResponse"]
    F639["UpdateConfigurationResponse.String"]
    F640["UpdateConfigurationResponse.LatestAppliedVersion"]
    F641["UpdateConfigurationResponse.SetLatestAppliedVersion"]
    F642["UpdateConfigurationResponse.Err"]
    F643["UpdateConfigurationResponse.HasErr"]
    F644["UpdateConfigurationResponse.ErrBytes"]
    F645["UpdateConfigurationResponse.SetErr"]
    F646["NewUpdateConfigurationResponse_List"]
    F647["UpdateConfigurationResponse_List.At"]
    F648["UpdateConfigurationResponse_List.Set"]
    F649["UpdateConfigurationResponse_List.String"]
    F650["UpdateConfigurationResponse_Promise.Struct"]
    F651["ConfigurationManager.UpdateConfiguration"]
    F652["ConfigurationManager_ServerToClient"]
    F653["ConfigurationManager_Methods"]
    F654["NewConfigurationManager_updateConfiguration_Params"]
    F655["NewRootConfigurationManager_updateConfiguration_Params"]
    F656["ReadRootConfigurationManager_updateConfiguration_Params"]
    F657["ConfigurationManager_updateConfiguration_Params.String"]
    F658["ConfigurationManager_updateConfiguration_Params.Version"]
    F659["ConfigurationManager_updateConfiguration_Params.SetVersion"]
    F660["ConfigurationManager_updateConfiguration_Params.Config"]
    F661["ConfigurationManager_updateConfiguration_Params.HasConfig"]
    F662["ConfigurationManager_updateConfiguration_Params.SetConfig"]
    F663["NewConfigurationManager_updateConfiguration_Params_List"]
    F664["ConfigurationManager_updateConfiguration_Params_List.At"]
    F665["ConfigurationManager_updateConfiguration_Params_List.Set"]
    F666["ConfigurationManager_updateConfiguration_Params_List.String"]
    F667["ConfigurationManager_updateConfiguration_Params_Promise.Struct"]
    F668["NewConfigurationManager_updateConfiguration_Results"]
    F669["NewRootConfigurationManager_updateConfiguration_Results"]
    F670["ReadRootConfigurationManager_updateConfiguration_Results"]
    F671["ConfigurationManager_updateConfiguration_Results.String"]
    F672["ConfigurationManager_updateConfiguration_Results.Result"]
    F673["ConfigurationManager_updateConfiguration_Results.HasResult"]
    F674["ConfigurationManager_updateConfiguration_Results.SetResult"]
    F675["ConfigurationManager_updateConfiguration_Results.NewResult"]
    F676["NewConfigurationManager_updateConfiguration_Results_List"]
    F677["ConfigurationManager_updateConfiguration_Results_List.At"]
    F678["ConfigurationManager_updateConfiguration_Results_List.Set"]
    F679["ConfigurationManager_updateConfiguration_Results_List.String"]
    F680["ConfigurationManager_updateConfiguration_Results_Promise.Struct"]
    F681["ConfigurationManager_updateConfiguration_Results_Promise.Result"]
    F682["CloudflaredServer.RegisterUdpSession"]
    F683["CloudflaredServer.UnregisterUdpSession"]
    F684["CloudflaredServer.UpdateConfiguration"]
    F685["CloudflaredServer_ServerToClient"]
    F686["CloudflaredServer_Methods"]
    F687["init"]
    F73 --> F73
    F82 --> F345
    F113 --> F73
    F159 --> F160
    F175 --> F57
    F189 --> F22
    F212 --> F115
    F271 --> F57
    F285 --> F128
    F312 --> F57
    F326 --> F22
    F381 --> F350
    F407 --> F421
    F411 --> F411
    F475 --> F476
    F484 --> F456
    F493 --> F374
    F508 --> F398
    F572 --> F573
    F604 --> F554
    F652 --> F653
    F675 --> F636
    F685 --> F686
```

## Accuracy Notes

- Generated from Go AST parsing and source text pattern extraction.
- Source link is authoritative for disputed semantics; keep this atom synchronized with the linked file.

## Rust Porting Notes

- **Schema compilation**: The `.capnp` schema file compiles to Go via `capnpc-go` → compile to Rust via `capnpc-rust` in a `build.rs` script using `capnpc::CompilerCommand`.
- **Generated code volume**: 300+ generated Go entry points → Rust `capnp` codegen produces equivalent reader/builder types; do not hand-write these.
- **RPC interfaces**: `TunnelServer`, `RegistrationServer`, `SessionManager`, `ConfigurationManager` Cap'n Proto interfaces → generate Rust server traits and client stubs automatically.
- **Schema compatibility**: The `.capnp` schema is the wire-format source of truth — the Rust port must use the **identical schema file** to ensure wire compatibility with the Go edge.
- **Promise pipeline**: Go Cap'n Proto uses promise pipelining → Rust `capnp-rpc` supports the same; use `request.send().pipeline` for pipelined calls.
- **Quirk — generated code is read-only**: Do not modify the generated Rust output; all customization goes in the pogs wrapper layer (see [atoms/tunnelrpc/pogs/registration_server](../pogs/registration_server.md) and [atoms/tunnelrpc/pogs/session_manager](../pogs/session_manager.md)).
