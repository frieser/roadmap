# Backend Track — Schematic Summary

## Internet
| Area | Core |
|---|---|
| HTTP | Stateless. GET/POST/PUT/PATCH/DELETE. HTTP/2: multiplex. HTTP/3: QUIC. |
| DNS | Root→TLD→Authoritative. Records: A, AAAA, CNAME, MX, TXT. TTL→cache. |
| Hosting | Shared→VPS→Dedicated→Cloud. IaaS/PaaS/SaaS/Serverless. |

## Git
Blob→Tree→Commit→Tag. Workflows: Gitflow (release cycles), Trunk-based (CI/CD), GitHub Flow (PR→main). Advanced: reflog, bisect, filter-repo. Perf: shallow/partial clone, LFS. Go: `go-git`.

## Relational Databases
| DB | Arch | Go Driver | Key |
|---|---|---|---|
| PostgreSQL | Process-per-conn, MVCC, WAL | `pgx/v5` | Extensibility (PostGIS, pgvector) |
| MySQL | Thread-per-conn, InnoDB | `go-sql-driver` | Read-perf |
| MariaDB | MySQL fork + Aria | same as MySQL | FOSS guarantee |
| SQLite | Embedded, file-based | `modernc.org/sqlite` | Zero-config |
| MS SQL | Thread-based, SQLOS | `go-mssqldb` | SSMS, Always Encrypted |
| Oracle | Process-based, SGA/PGA, RAC | `godror`/`go-ora` | RAC, PL/SQL |

## NoSQL
| Type | DB | CAP | Go Driver |
|---|---|---|---|
| Document | MongoDB | CP | `mongo-driver` |
| Document | CouchDB | AP | `kivik` |
| Document | RethinkDB | CP | `rethinkdb-go` |
| KV | Redis | CP | `go-redis/v9` |
| KV | DynamoDB | AP | `aws-sdk-go-v2` |
| TS | InfluxDB | - | `influxdb-client-go` |
| TS | TimescaleDB | - | `pgx/v5` |
| Column | Cassandra | AP | `gocql` |
| Column | HBase | CP | `gohbase` |
| Graph | Neo4j | CP | `neo4j-go-driver` |
| Graph | AWS Neptune | - | `gremlin-go` |

## API Styles
| Style | Protocol | Data | Key |
|---|---|---|---|
| REST | HTTP | JSON/XML | Resource-oriented |
| gRPC | HTTP/2 | Protobuf | Streaming, code gen |
| GraphQL | HTTP | Schema | Client-specified fields |
| SOAP | HTTP/SMTP | XML/WSDL | WS-Security, ACID |
| OpenAPI | YAML/JSON | Spec | Docs, code gen, mocking |

### Auth
Basic (legacy), JWT (stateless, RS256/HS256), Cookie/Session (stateful), OAuth2 (delegated), OIDC (ID Token), SAML (enterprise SSO).

## Caching
Server: Redis (persistence, structures) / Memcached (multithreaded, volatile). CDN: edge TTL, purge. Client: Cache-Control, ETag, 304.

## Web Security
| Topic | Key |
|---|---|
| HTTPS | TLS 1.2/1.3, asymmetric→symmetric |
| Hashing | MD5/SHA-1 broken. SHA-2/3 fast (not for passwords). |
| Passwords | bcrypt (cost factor), scrypt (memory-hard), Argon2 |
| OWASP | Injection, Broken Access Control, SSRF, Crypto Failures |
| CORS | Browser, simple vs preflight (OPTIONS) |
| CSP | Header blocking XSS |

## Testing
| Type | Scope | Go Tools |
|---|---|---|
| Unit | Single fn | `testing`, table-driven, `testify` |
| Integration | Components | Build tags, Testcontainers |
| Functional | Endpoint | `httptest.NewRecorder` |

## Database Deep Dive
ORM: GORM (reflection), Ent (code-gen). ACID: `db.BeginTx()`. N+1: eager load `Preload`. Indexes: B-Tree/Hash/GIN, left-prefix rule. Sharding: key-hash/range. Replication: single/multi-leader. CAP: CP (Mongo/HBase/Redis) vs AP (Cass/Couch/Dynamo). Failures: circuit breaker, backoff, context timeouts.

## Architecture
| Pattern | Go |
|---|---|
| Singleton/Strategy | `sync.Once`, interfaces |
| DDD | Value Objects, Entities, Aggregates, Repository |
| Monolith | Modular `internal/` packages |
| Microservices | Go-kit, gRPC inter-service |
| Serverless | FaaS, fast cold starts |
| Service Mesh | Sidecar, mTLS, Istio/Linkerd |
| 12-Factor | Config in env, stateless |

## Web Servers
Nginx (event-driven, reverse proxy). Apache (process/thread, `mod_proxy`). Caddy (Go, auto-HTTPS, extendable). IIS (app pools, HttpPlatformHandler). Tomcat (servlet container, Java).

## Search & Messaging
ES (inverted index, shards). Solr (faceted). RabbitMQ (AMQP, direct/topic/fanout). Kafka (commit log, partitions, consumer groups). Go: `amqp091-go`, `kafka-go`, `go-elasticsearch`.

## Containerization
VM (hypervisor). Container (namespaces+cgroups). LXC (system container). Docker: Overlay2, CoW. K8s: control plane + workers, CNI. Go: `client-go`, `kubebuilder`.

## Real-Time
Short Polling (HTTP periodic), Long Polling (HTTP held), SSE (S→C), WebSocket (full-duplex). Go: `gorilla/websocket` vs `coder/websocket`.

## Scale
Graceful Degradation, Throttling (Token/Leaky, `golang.org/x/time/rate`), Backpressure (bounded queue), Circuit Breaker (Closed→Open→Half-Open, `sony/gobreaker`), Strangler Fig/Blue-Green/Canary. Vertical vs Horizontal scaling.

## Observability
Metrics: `prometheus/client_golang`. Logs: `zap`/`zerolog`. Traces: `opentelemetry-go`. RED for services, USE for infra. Context propagation via `context.Context`.
