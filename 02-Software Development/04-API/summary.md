# API Summary — Design & Security

## API Design
REST — Architectural style. 6 constraints: Client-Server, Stateless, Cacheable, Uniform Interface, Layered System, Code-on-Demand. Resources = nouns, HTTP verbs = actions. HATEOAS at Level 3 (Richardson). Stateless = no server session.
Simple JSON APIs — Pragmatic REST. Raw objects vs envelopes (metadata wrapper). Content-Type: application/json. omitempty for clean payloads.
SOAP — Protocol, XML-only. WSDL contract. WS-Security end-to-end. ACID transactions. Heavy enterprise (banking, telecom).
GraphQL — Query language. Single endpoint. Client selects fields → no over/under-fetching. Schema-first (gqlgen) or code-first. Dataloaders fix N+1. Evolution without versioning (@deprecated).
gRPC — RPC over HTTP/2 + Protobuf. Binary, multiplexing, bidirectional streaming. Unary/Server/Client/Bidi. .proto contract → protoc generates stubs.
WebSockets — Full-duplex over TCP. HTTP upgrade→101. Ping/pong keepalives. Hub pattern (gorilla/websocket). Scale with Pub/Sub.
SSE — Unidirectional server→client. text/event-stream. http.Flusher. Auto-reconnect + Last-Event-ID. HTTP/2 multiplexing fixes connection limit.
Versioning — URI (/v1/), Header, Query. URI most popular. GraphQL: evolution. Sunset header for deprecation.
Pagination — Offset (simple, slow at scale), Cursor (keyset, constant-time, stable), Page-based. Link header (RFC 5988). X-Total-Count.
Error Handling — Correct HTTP codes. Never 200 for errors. Consistent JSON payload. Never expose traces. RFC 7807 (application/problem+json): type, title, status, detail, instance.

## API Security
AuthN vs AuthZ — Authentication = who. Authorization = what. Basic Auth: base64 creds, HTTPS mandatory, no logout — avoid. Token-based: Bearer token, stateless, scalable, short lifespan + refresh tokens.
JWT — Header.Payload.Signature. HS256/RS256. Min 256-bit secret. Never store secrets in payload (base64 ≠ encrypted). Short TTL (15min). Never extract alg from header — whitelist. None attack: alg=none bypasses. Key confusion: swap RS256→HS256, sign with public key. Go: golang-jwt/jwt/v5, check token.Method in Keyfunc.
OAuth 2.0 — Authorization framework. Auth Code Flow (server-side). PKCE for SPAs. Implicit deprecated. State param prevents Login CSRF. Exact-match redirect_uri (no wildcards). OIDC adds ID token. Go: golang.org/x/oauth2.
RBAC — User→Role→Permission. Casbin: model.conf + policy. Check permissions, not roles. Role inheritance (g=_,_). Avoid role explosion.
ABAC — Subject+resource+action+environment. OPA+Rego. default allow=false. PEP middleware → PDP. Prevents role explosion.
CORS — Browser-side. SOP blocks cross-origin. Access-Control-Allow-Origin. Preflight (OPTIONS) for non-simple. AllowCredentials=true → no wildcard. Go: rs/cors.
Rate Limiting — Token Bucket (bursts), Leaky, Fixed Window (boundary bug), Sliding Window. Distributed: Redis. 429+Retry-After. Go: x/time/rate, tollbooth.
Input Validation — Allowlist > blocklist. Parameterized queries → no SQLi. Go: go-playground/validator. HTML: bluemonday.
Output — X-Content-Type-Options: nosniff, X-Frame-Options: DENY, CSP. Remove Server/X-Powered-By. Never return sensitive data. Force Content-Type.
CI/CD — Audit tests, code review, SAST, dependency scan, rollback deploys.

## REST vs GraphQL vs gRPC
| Feature | REST | GraphQL | gRPC |
|---|---|---|---|
| Style | Resource-oriented | Query language | RPC |
| Transport | HTTP/1.1 | HTTP/1.1 | HTTP/2 |
| Format | JSON/XML | JSON | Protobuf |
| Endpoints | Multiple | Single | Single |
| Streaming | No (SSE) | Subscriptions (WS) | Native bidirectional |

## Go Patterns
net/http — ServeMux method+path (Go 1.22+). chi sub-routing. Handler structs hold deps. json.NewEncoder/Decoder. Middleware chain.
gqlgen — Schema-first: .graphql → generate resolvers. Dataloaders batch fetch. graphql-go: code-first alternative.
grpc-go — protoc → pb + _grpc.pb.go. Impl server interface. net.Listen + grpc.NewServer + RegisterService + s.Serve.
JWT — golang-jwt/jwt/v5: NewWithClaims+SignedString, Parse+Keyfunc checks token.Method type assert.
Validation — validator struct tags. Casbin (RBAC). OPA/Rego (ABAC). bcrypt/argon2. crypto/rand + subtle.ConstantTimeCompare.
WebSocket — gorilla/websocket. Upgrader+CheckOrigin. Hub: register/unregister/broadcast chans. readPump+writePump goroutines.
Rate Limiting — x/time/rate (Token Bucket). tollbooth (per-IP). Redis for distributed.
Idempotency — Idempotency-Key header. Redis SETNX lock. Cache response. 409 concurrent. 422 key+different body.

## API Rules
1. Nouns for URIs, HTTP methods for verbs — /users not /getUsers.
2. Never trust client input — validate, parameterize, allowlist.
3. Never expose stack traces — log internally, return generic + request_id.
4. Always use HTTPS — HSTS, secure ciphers, no mixed content.
5. Expire tokens short, rotate secrets, never trust alg header.
6. Exact-match redirect_uri in OAuth — no wildcards.
7. Rate limit per-user/IP with distributed counters (Redis) — not in-memory.
8. Version APIs explicitly — URI versioning pragmatic, GraphQL evolves.
