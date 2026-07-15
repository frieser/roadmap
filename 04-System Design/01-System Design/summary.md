# System Design - Compact Schematic
## 01 Introduction
| Metric | Unit | Goal | Go |
|:---|:---|:---|:---|
| Latency | ms/req | Minimize | `context.WithTimeout` |
| Throughput | RPS/QPS | Maximize | buffered chan |
| Scalability | capacity/load | Horizontal | stateless handlers |
HLD=blocks+flow / LLD=classes+code. Framework: scope→est→API→schema→HLD→deep dive→bottlenecks

## 02 CAP Theorem
| Choice | Partition behavior | Example |
|:---|:---|:---|
| AP | Both sides write, eventual | Cassandra, DynamoDB |
| CP | Minority stops, reject writes | Etcd, MongoDB (default) |
**Consistency**: Strong(linearizable,banking)→Causal(order-preserving)→Eventual(converges,DNS)→Weak(best-effort,VoIP)
**Reconciliation**: LWW(clock-skew) | Vector Clocks | CRDTs(math converge)

## 03 Availability Patterns
| Failover | Active | Complexity | Util |
|:---|:---|:---|:---|
| Active-Active | All serve | High(sync) | 100% |
| Active-Passive | One+standby | Low(heartbeat) | Idle waste |
**Replication**: Master-Slave(1 writer,N readers,lag) | Multi-Master(N writers,conflict)
**Avail Math**: Serial=A×B×C(degrades) | Parallel=1-(1-A)²(improves) | Nines: 99.9%=8.76hr, 99.99%=52min, 99.999%=5.26min

## 04 CAP Deep
P→CP(correctness) or AP(uptime). No P→Latency(eventual) or C(sync-rep). **PACELC**. Mapping: Banking→CP, Social→AP, DNS→AP, Trading→CP

## 05 CDN
| | Pull CDN | Push CDN |
|:---|:---|:---|
| Fill | Cache miss→origin fetch | Explicit upload(CI/CD) |
| 1st req | Slow | Pre-warmed |
| Use | Websites, UGC, variable | Large files, VoD, critical |

## 06 Load Balancers
| | L4(Transport) | L7(Application) |
|:---|:---|:---|
| Route by | IP+port | URL, headers, cookies |
| Speed | Fast, low CPU | Slower, inspect content |
| Ex | AWS NLB | NGINX, AWS ALB |
**Algos**: RR(uniform) | WeightedRR(mixed HW) | IP Hash(sticky) | LeastConn(variable) | LeastResp(latency) | ConsistentHash(caching)

## 07 Application Layer
Microservices: independent deploy, per-service scale. Cost: net latency, dist data, test complexity. **Go**: Go-kit, Go-micro, gRPC. **Service Disc**: Client-side(direct) vs Server-side(LB,+hop). Tools: Consul, Etcd, ZK, K8s

## 08 Databases
| SQL Tech | Solves | Trade-off |
|:---|:---|:---|
| Indexing | Slow queries | Write overhead |
| Sharding | Horizontal scale | Cross-shard ops expensive |
| Denormalization | Read speed | Write complexity |
| Replication | Read scale, HA | Lag |
| NoSQL | Model | Example | Use |
|:---|:---|:---|:---|
| KV | Hash map | Redis, DynamoDB | Caching, sessions |
| Document | JSON/BSON | MongoDB | CMS, catalogs |
| Wide-Column | Column fam | Cassandra | Time-series, IoT |
| Graph | Nodes+edges | Neo4j | Social, fraud |
**SQL vs NoSQL**: SQL=ACID+vertical | NoSQL=BASE+horizontal. Use both (polyglot)

## 09 Caching
| Strategy | Write | Read | Risk |
|:---|:---|:---|:---|
| Cache-Aside | DB+invalidate | Miss→DB→cache | 1st req slow |
| Write-Through | Cache+DB sync | Always hit | Write latency |
| Write-Behind | Cache then DB async | Fast writes | Data loss |
| Refresh-Ahead | Proactive refresh | Always fresh | DB waste |
Layers: Client→CDN→WebServer→DB→App-local→App-distributed(Redis). **Go**: `sync.Map`(local), `singleflight`(stampede), `map+RWMutex`(general)

## 10 Asynchronism
| Tool | Model | Throughput | Use |
|:---|:---|:---|:---|
| RabbitMQ | Smart broker | 10K/s | Routing, tasks |
| Kafka | Log-based | Millions/s | Streaming, replay |
| Redis(asynq) | KV-backed | Very fast | Simple queues |
**Patterns**: Event-driven(state changes) | Schedule-driven(cron) | Background jobs(user action)
Back Pressure: Buffer(OOM) | Drop(load shed) | Throttle(TCP). Go unbuffered chan=natural BP

## 11 Communication
| Protocol | Reliability | Use | API | Serial | Best For |
|:---|:---|:---|:---|:---|:---|
| TCP | Reliable ordered | HTTP, SMTP | REST | JSON | Public APIs |
| UDP | None | VoIP, gaming | GraphQL | JSON | Complex data, mobile |
| HTTP/3(QUIC) | Multiplexed | Modern web | gRPC | Protobuf | Internal, streaming |
Idempotency: GET/PUT/DELETE safe. POST→use `Idempotency-Key` to prevent duplicates.

## 12 Performance Antipatterns
| Antipattern | Problem | Go Fix |
|:---|:---|:---|:---|
| Improper Instantiation | Exhaustion | Share `sql.DB`, `http.Client`, `sync.Pool` |
| Monolithic Persistence | Coupling | DB-per-service, polyglot |
| Chatty I/O | N+1 | Batch IN(), DataLoader, bufio |
| Retry Storm | DDoS failing service | Exp backoff+jitter, circuit breaker |
| Busy Database | DB doing logic | Move logic to Go app |

## 13 Monitoring
**K8s Probes**: Liveness(restart, internal only) | Readiness(rm from LB, check deps) | Startup(delay checks)
**4 Golden Signals**: Latency(p99) | Traffic(RPS) | Errors(5xx) | Saturation(CPU, goroutines)
**OTel Pillars**: Metrics(alerting) | Logs(structured) | Traces(request flow). **Go**: `log/slog`, `prometheus/client_golang`, `otel`

## 14 Cloud Design Patterns
| Pattern | Category | Key Idea |
|:---|:---|:---|
| Circuit Breaker | Availability | Closed→Open→Half-Open, fail-fast |
| Health Endpoint | Availability | `/healthz`(liveness) + `/readyz`(readiness) |
| CQRS | Data | Separate read/write models, scale independently |
| Event Sourcing | Data | Append-only log, replay→state |
| Pub-Sub | Messaging | Fan-out, decouple producers/consumers |
| Priority Queue | Messaging | Sort by importance, aging prevents starvation |
CQRS+ES: projections from event stream. Snapshotting prevents full replay.

## 15 Big Data
Hadoop MR: HDFS(disk), slow batch. Spark: RAM+DAG, 100x faster. Cloud-native: BigQuery, Snowflake.
Batch=min-hr(ETL,audit) vs Stream=ms-sec(fraud,dashboards). Go goroutines+chans→MapReduce sim.

## Quick Picks
DB: SQL(PG)→KV(Redis)→Doc(Mongo)→WideCol(Cass)→Graph(Neo4j)
API: REST(public)→gRPC(internal)→PubSub(events)→Queue(background)
Cache: Read-heavy→CacheAside, Write-heavy→WriteBehind, Hotkeys→RefreshAhead
