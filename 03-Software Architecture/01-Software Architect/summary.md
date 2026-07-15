# Software Architect — Compact Summary

## Responsibilities
- **Tech decisions**: Build vs Buy. Core differentiator → build. Commodity → buy. Boring tech for stability.
- **Architecture design**: ADRs (context/decision/consequences). Interface-driven boundaries. Trade-off analysis.
- **Requirements elicitation**: NFRs > functional. Quantify: "10K TPS at p99 <200ms" not "must be fast".
- **Documentation**: C4 model (Context→Container→Component→Code). Docs as Code in Git. ADRs capture "why".
- **Standards enforcement**: Automate via CI. golangci-lint, gosec, pre-commit hooks. Not in CI → not a standard.
- **Collaboration**: API-first design. RFC process. Shared contracts (OpenAPI, Protobuf) before code.
- **Coach/consult**: Force multiplier. Guardrails not gates. Code reviews = teaching moments.

## Skills
- **Design architecture**: Styles (Monolithic, Layered, Microservices, Event-Driven, Hexagonal). ATAM reviews. C4 visualization.
- **Decision-making**: Last Responsible Moment. Trade-off analysis. ATAM, QAW workshops. RFC consensus.
- **Simplify**: Essential complexity (domain) vs Accidental (tools). KISS, YAGNI, modular monolith first. DDD bounded contexts.
- **Coding**: Stay hands-on. 50/50 coding/design. Prototypes vs production rigor. Lead by example.
- **Balance**: Tactical vs Strategic. Innovation tokens (≤2 new tech/project). Paved roads. Architecture Runway.
- **Communication**: Translate tech→business value. Active listening. Whiteboarding. Async (RFC/ADR) vs Sync (conflict resolution).
- **Estimation**: T-shirt sizing (XS→XXL). PoCs de-risk critical decisions. Build vs Buy TCO analysis.
- **Marketing**: Sell decisions via data. Platform as product. Internal branding. Cialdini persuasion principles.
- **Consult/coach**: Influence without authority. Mentoring. Idiomatic code guidance. Make yourself redundant.

## Architecture Styles

| Style | Essence | When |
|-------|---------|------|
| Monolithic | Single deployable unit. Simple start. | New domains, small teams. |
| Layered (N-Tier) | Horizontal separation (UI→Service→Data). | Enterprise apps, separation of concerns. |
| Microservices | Independent deploy by business capability. DB per service. | Scale, team autonomy, complex domains. |
| SOA | Enterprise reuse via ESB. Heterogeneous protocols. | Legacy integration, banking. |
| Event-Driven | Async Pub/Sub. Temporal decoupling. | High throughput, loose coupling. |
| Serverless | FaaS + BaaS. Pay-per-use. Stateless. | Variable workloads, event triggers. |
| Hexagonal | Ports & Adapters. Core isolated from infra. | Testability, tech-swap flexibility. |
| CQRS | Separate read/write models. Event Sourcing. | Complex queries + strict writes. |
| Reactive | Responsive, Resilient, Elastic, Message-Driven. | Real-time, high concurrency. |
| Client-Server | Foundational. 2-tier→3-tier→N-tier evolution. | Web/mobile backends. |

## Patterns & Design
- **SOLID**: SRP (one reason to change), OCP (extend, don't modify), LSP (substitutable), ISP (small interfaces), DIP (depend on abstractions).
- **TDD**: Red→Green→Refactor. Forces testable architecture. Living documentation.
- **DDD**: Ubiquitous language. Bounded contexts. Aggregates. Rich domain model over anemic.
- **CQRS**: Command (write, normalized) vs Query (read, denormalized). Event Sourcing as source of truth.
- **Saga**: Sequence of local transactions. Choreography (events) or Orchestration (coordinator). Compensating txns on failure.
- **Strangler Fig**: Proxy in front of legacy. New features→new services. Incrementally strangle old system.
- **Anti-Corruption Layer**: Adapter between bounded contexts. Translates models. Prevents legacy leakage.
- **OOP patterns**: Strategy (swappable implementations), Factory (DI), Adapter (Ports & Adapters). Composition over inheritance.
- **Actor Model**: Async message passing. Supervision trees. "Let it crash". vs Go CSP: channels, synchronous.
- **FP concepts**: Pure functions, immutability, higher-order functions. Side effects at edges.

## Security
- **Threat modeling**: STRIDE/PASTA during design. Shift left. Deny by default.
- **OWASP Top 10 (2025)**: Broken Access Control, Security Misconfig, Supply Chain (SBOM), Crypto Failures, Injection, Insecure Design, Auth Failures, Data Integrity, Logging Failures, Exceptional Conditions.
- **Auth patterns**: JWT (stateless, scalable). OAuth2+OIDC (delegated identity). PKCE for SPAs. MFA mandatory. SSO (SAML/OIDC).
- **Hashing**: Argon2id (password, memory-hard) > Bcrypt (legacy) > SHA-256 (integrity only, fast). Salt + Pepper.
- **PKI/mTLS**: Asymmetric crypto. Chain of trust (Root→Intermediate→Leaf). mTLS for Zero Trust in mesh.
- **Crypto**: AES-GCM for data. TLS 1.3 minimum. HSTS. HttpOnly + Secure cookies. KMS/HSM for keys.

## Data
- **SQL**: ACID. Isolation levels (Read Committed→Serializable). B-Tree/Hash/GIN indexes. Sharding last resort. NewSQL (CockroachDB, TiDB).
- **NoSQL types**: Document (MongoDB), Key-Value (Redis), Column-Family (Cassandra), Graph (Neo4j). Polyglot persistence.
- **CAP theorem**: Choose CP (consistency) or AP (availability). P is mandatory. PACELC extends to normal ops.
- **ETL→ELT**: Cloud warehouses (Snowflake, BigQuery). dbt for transformations. Orchestrators (Airflow, Dagster). Data Lakehouse (Iceberg, Delta).
- **Big Data**: Hadoop (MapReduce, HDFS, YARN) → Spark (in-memory, DAGs, 100x faster). Batch vs Stream.

## Web/Mobile
- **Rendering**: CSR/SPA (interactive), SSR (SEO, fast FCP), SSG (static, CDN), ISR (hybrid). Edge rendering for latency.
- **Micro-Frontends**: Run-time integration (Module Federation, Web Components). Custom events for communication. Avoid without 3+ teams.
- **PWA**: Service Workers (offline, cache), Web Manifest (installable). Polyfills/transpilation for compatibility.
- **Frameworks**: React (de facto), Vue (simpler), Angular (enterprise). HTMX for backend-heavy teams. Native (Swift/Kotlin) vs cross-platform (Flutter, RN).

## APIs & Integration
- **REST**: 6 constraints. Richardson Maturity (Level 0→3). HATEOAS. Stateless for horizontal scaling. Idempotency (GET/PUT/DELETE).
- **GraphQL**: Single endpoint. Client-specified queries. Prevents over/under-fetching. N+1 solved with Dataloaders. Schema-first (gqlgen) or code-first.
- **gRPC**: Protobuf + HTTP/2. Binary, streaming (unary/server/client/bidirectional). Interceptors = middleware. gRPC-Web for browsers.
- **API Gateway**: North-South traffic. Auth, rate limiting, routing. vs Service Mesh (East-West: mTLS, retries, circuit breaking).
- **Message queues**: Point-to-Point vs Pub/Sub. RabbitMQ (smart broker), Kafka (log-based, replay), Redis Streams (lightweight). ACKs, DLQ.
- **Orchestration**: BPEL (XML legacy) → Workflow-as-Code (Temporal, Camunda). Choreography vs Orchestration. Saga pattern.
- **SOA/ESB**: Smart pipes, dumb endpoints. Centralized routing/transformation. Microservices reversed this: smart endpoints, dumb pipes.

## Networks
- **OSI (7-layer)**: Application→Presentation→Session→Transport→Network→Data Link→Physical. Encapsulation/decapsulation.
- **TCP/IP (4-layer)**: Application→Transport (TCP/UDP)→Internet (IP)→Network Interface. Practical internet model.
- **HTTP versions**: 1.1 (persistent, HOL blocking), 2 (multiplexing, binary, HPACK), 3 (QUIC/UDP, no TCP HOL, connection migration).
- **TLS 1.3**: 1-RTT handshake. 0-RTT resume. Mandatory in HTTP/3. Certificates (X.509), OCSP stapling.
- **Proxies**: Forward (client privacy), Reverse (SSL termination, LB, caching). L4 (IP/port) vs L7 (HTTP headers). Nginx, HAProxy, Envoy.
- **Firewalls**: Stateless packet filter → Stateful → Proxy/WAF (L7). Cloud: Security Groups (stateful), NACLs (stateless). DMZ for public services.

## Operations
- **IaC**: Declarative (Terraform, Pulumi) over imperative. Immutable infra. Idempotency. Drift detection.
- **Cloud providers**: AWS (breadth), Azure (enterprise/MS stack), GCP (data/AI/K8s). Multi-cloud adds complexity. Hybrid for gradual migration.
- **Serverless**: FaaS (Lambda) + BaaS. Cold starts mitigated by Go/Rust, provisioned concurrency. Stateless. Double billing risk.
- **Linux**: Kernel vs User space. Everything is a file. Signals (SIGTERM=15 graceful, SIGKILL=9 forced). Containers = namespaces + cgroups.
- **Service Mesh**: Sidecar pattern (Envoy/Linkerd). Data Plane vs Control Plane. mTLS, traffic splitting, circuit breaking. Ambient mesh (sidecarless).
- **CI/CD**: CI (merge+test frequently) → Continuous Delivery (manual deploy) → Continuous Deployment (auto). Blue/Green, Canary, Rolling updates.
- **Containers**: Image (layered, OCI) vs Container (runtime, writable layer). Namespaces (isolation), cgroups (limits). Docker (single host), K8s (orchestration).
- **Cloud patterns**: Circuit Breaker (stop cascading failures), Retry+Backoff, Sidecar/Ambassador, Strangler Fig, CQRS, Event Sourcing.

## Enterprise
- **TOGAF**: ADM cycle (Prelim→A-H phases). ABB (abstract) vs SBB (concrete). Architecture Repository. Gold standard EA framework.
- **IAF**: Capgemini 4x4 matrix (Contextual/Conceptual/Logical/Physical × Business/Info/IS/Tech). Artifact-centric.
- **BABOK**: 6 Knowledge Areas. BACCM (Change/Need/Solution/Stakeholder/Value/Context). Bridge business↔tech.
- **PMI/PMBOK**: 5 Process Groups (Initiating→Planning→Executing→M&C→Closing). Project scope/schedule.
- **ITIL**: Service lifecycle (Strategy→Design→Transition→Operation→CSI). Focus on operability.
- **RUP**: 4 phases (Inception→Elaboration→Construction→Transition). Iterative. Elaboration = architecture baseline.
- **PRINCE2**: 7 Principles. Manage by stages. Continued business justification. PRINCE2 Agile variant.
- **Agile**: Kanban (flow, WIP limits, architectural runway). Scrum (sprints, DoD, architect works ahead). XP (TDD, pair programming, YAGNI). SAFe (ART, explicit architect roles). LeSS (feature teams over component teams).
- **UML**: Structure (Class, Component, Deployment) + Behavior (Sequence, Activity, State Machine). Mermaid for code-based diagrams.
- **Enterprise platforms**: MS Dynamics 365 (Dataverse/OData). SAP S/4HANA (HANA in-memory, ABAP, BTP). Documentum (ECM/DQL). IBM BAW (BPMN, BPEL). Salesforce (multi-tenant, Apex, SOQL).

## Architect Rules
1. Everything is a trade-off. If no downside, look harder.
2. NFRs dictate architecture, not features.
3. Design for testability. Hard to test = bad architecture.
4. ADRs capture "why." Without them, decisions will be re-litigated.
5. Stay in the IDE. Ivory tower architects lose context.
6. Choose boring technology. Limit innovations per project.
7. Build what differentiates. Buy everything else.
8. Design stateless. Horizontal scaling demands it.
9. Automate standards. Manual enforcement doesn't scale.
10. The best architecture is the one the team can actually build.