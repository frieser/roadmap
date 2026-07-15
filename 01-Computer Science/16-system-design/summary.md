# System Design — Summary

## CAP Theorem
- **C**onsistency: all nodes see same data. Sync replication. CP = banking.
- **A**vailability: every request gets response. May be stale. AP = social.
- **P**artition Tolerance: system survives network cuts. Must choose CP or AP.

## Scaling
- **Vertical**: bigger server. Simple, hard ceiling, SPOF. Small DBs.
- **Horizontal**: more servers. Infinite scale, fault tolerant. Needs LB.
- DB is bottleneck. Fix: sharding, read replicas, caching layer.

## Caching Strategies
- Cache-Aside: check cache → miss → read DB → populate cache. Common.
- Write-Through: write cache → cache writes DB sync. Safe, slower write.
- Write-Back: write cache → cache writes DB async. Fast, risk data loss.
- Write-Around: write DB directly, cache on read miss. Write-once data.

## Patterns
| Pattern | Problem | Solution |
|---------|---------|----------|
| Circuit Breaker | Repeated calls to failing service | Trip after N failures, fail fast |
| Bulkhead | Slow service exhausts shared thread pool | Isolate pools per service |
| Sidecar | Cross-cutting concerns per container | Deploy helper alongside main container |
| Ambassador | Consumer-side network complexity | Proxy handles retry, routing, monitor |
| Exponential Backoff | Retries overwhelm recovering service | Wait 1s→2s→4s between retries |
| Saga | Distributed transactions across services | Compensating steps for rollback |
| CQRS | Read/write contention on same model | Separate read and write data stores |

## Load Balancing
- L4 (TCP): fast, IP/port. L7 (HTTP): URL/cookie routing, SSL termination.
- Algos: Round Robin, Least Connections, IP Hash (sticky sessions).

## Clustering
- Active-Active: all serve. Active-Passive: standby failover.
- Heartbeat + Quorum (majority vote) prevents Split Brain.

## API Styles
| Style | Transport | Strengths | Weaknesses |
|-------|-----------|-----------|------------|
| REST | HTTP/1.1 | Cacheable, stateless, uniform | Over/under-fetching |
| GraphQL | HTTP/POST | Exact fields, single endpoint | N+1, caching hard |
| gRPC | HTTP/2 | Binary, streaming, strict contracts | No browser support |

## Real-time Protocols
| Protocol | Direction | Overhead | Use Case |
|----------|-----------|----------|----------|
| Short Poll | Client→Server | High | Rare updates |
| Long Poll | Client→Server | Medium | Near real-time fallback |
| SSE | Server→Client | Low | One-way feeds, notifications |
| WebSocket | Bi-directional | Lowest | Chat, gaming, trading |

## Proxy
- Forward: client side. VPN, anonymity, content filtering.
- Reverse: server side. LB, SSL termination, caching, DDoS protection.

## Message Queues
- Decouple producers/consumers. Backpressure. Async processing.
- Push (RabbitMQ) vs Pull (Kafka). Ack/Nack for reliability.

## CDN
- Edge servers near users. Pull Zone (cache on demand), Push Zone (upload).
- Invalidation: Purge or URL versioning (style.v2.css).

## Eviction
- LRU: remove least recently used. LFU: remove least frequently used.
- TTL: auto-expire after N seconds. Thundering Herd: lock on miss + jitter.

## Key Trade-offs
- CP vs AP: consistency or availability during partition.
- Push vs Pull queues: latency vs consumer control.
- Stateful (WS) vs Stateless (REST): real-time vs simplicity.
- Sync (Write-Through) vs Async (Write-Back): safety vs speed.
