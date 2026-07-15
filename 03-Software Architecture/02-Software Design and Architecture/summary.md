# Software Design & Architecture — Summary

## Clean Code
- **Naming**: Intention-revealing, pronounceable, proportional to scope. Avoid noise words (Info/Data), Hungarian notation.
- **Functions**: 5-10 lines, do one thing, single abstraction level. 0-2 args (3+ use struct). Guard clauses over nested ifs.
- **Comments**: Explain WHY, not WHAT. Valid: TODOs, warnings, godoc. Bad: redundant, commented-out code. Prefer renaming.
- **Pure Functions**: Deterministic, no side effects. Functional Core / Imperative Shell. Enables caching, parallelism, zero-mock tests.
- **Nulls/Bools**: Nil → empty collections, Null Object pattern. Bool args → split into named functions (`CreateAdmin`/`CreateGuest`).
- **Cyclomatic Complexity**: <10 per fn. Reduce via guard clauses, polymorphism, table-driven methods, extraction.
- **Framework Isolation**: Domain entities plain structs (no ORM tags). Dependency Rule: arrows point inward. Ports & Adapters.
- **Correct Constructs**: int for money (cents). map for lookups (not slice). Go set = `map[T]struct{}`. Know structure semantics.

## Programming Paradigms
- **Structured**: Sequence, selection, iteration. No GOTO. Single entry/exit. Foundation of all modern paradigms.
- **Functional**: Immutability, pure fns, higher-order fns, referential transparency. FP Core / Imperative Shell. Parse, Don't Validate.
- **OOP Paradigm**: Objects encapsulate data + behavior. Four pillars: encapsulation, abstraction, inheritance, polymorphism.

## OOP
- **Encapsulation**: Bundle data + methods. Unexported fields (Go: lowercase). Invariants enforced via methods, not public setters.
- **Abstraction**: Hide complexity behind interfaces. Go: small interfaces (Reader, Writer). Avoid leaky abstractions (ORM hides SQL perf).
- **Inheritance**: "Is-A." Go: struct embedding = composition, not subtyping. Diamond problem avoided. LSP: subtypes must be substitutable.
- **Polymorphism**: Many forms, one contract. Go: implicit structural typing (`interface{}`). Enables OCP. Duck typing at compile time.
- **Class Variants**: Abstract (partial blueprint, can't instantiate), Concrete (fully implemented), Interface (contract-only, no state).
- **Scope**: Go: capitalized = exported, lowercase = package-private. No class-level private (package boundary). Least privilege.

## Design Principles
- **SOLID**: SRP (one reason to change), OCP (extend via iface), LSP (substitutability), ISP (small focused ifaces), DIP (→ abstractions).
- **DRY**: Single source of truth for business knowledge. "Rule of Three" before abstraction. Accidental duplication = OK to repeat.
- **YAGNI**: Only what you need now. No speculative generality, no premature optimization. TDD enforces this.
- **Tell, Don't Ask**: Command the object; don't extract state and decide externally. Prevents anemic domain models.
- **Hollywood Principle**: "Don't call us, we'll call you." IoC — framework controls flow, calls your plugin/handler code.
- **Law of Demeter**: Only talk to immediate friends. `a.b().c().d()` violates. DTOs/fluent interfaces exempt (same object returned).
- **Composition over Inheritance**: Has-A > Is-A. Swappable at runtime, loose coupling. Go: only composition (embedding).
- **Encapsulate What Varies**: Identify volatile parts, isolate behind interfaces. Most patterns implement this (Strategy, Adapter, Factory).
- **Program to Abstractions**: Depend on interfaces, not concrete types. Enables DI + mocking. Stable stdlib types (string, time) exempt.

## Design Patterns — GoF 23

| Category | Pattern (Intent → Go Fit) |
|---|---|
| Creational | Factory Method (defer instantiation → Functional Options), Abstract Factory (families → iface factories), Builder (stepwise → Functional Options), Prototype (clone → Copy), Singleton (one instance → sync.Once/DI) |
| Structural | Adapter (convert iface → wrapper), Bridge (impl sep → iface composition), Composite (tree → recursive iface), Decorator (wrap → middleware), Facade (simplify → service layer), Flyweight (share → pooled), Proxy (surrogate → iface wrapper) |
| Behavioral | Chain (pass along → middleware), Command (capture → fn literal), Interpreter (grammar → AST), Iterator (traverse → range), Mediator (hub → event bus), Memento (snapshot → struct), Observer (1→many → channels), State (FSM → state iface), Strategy (swap algo → iface injection), Template (skeleton → base+hooks), Visitor (separate ops → type switch) |

**POSA (System-Level)**: Layers (abstraction tiers), Pipes & Filters (Go channels), Blackboard (AI/heuristic), Broker (gRPC/service mesh), MVC (interactive), Microkernel (core+plugins).

## Architectural Principles
- **Policy vs Detail**: Policy = business rules (stable); Detail = DB/UI/framework (volatile). Policy never depends on Detail.
- **Coupling & Cohesion**: Low coupling (modules independent via interfaces), High cohesion (related logic together). SRP drives cohesion.
- **Boundaries**: Lines separating policy from detail. Crossed via interfaces + DTOs. Partial boundaries = reserve option without full cost.

## Architectural Styles

| Style | Core Idea | When |
|---|---|---|
| Event-Driven | Async events via broker/mediator; producers & consumers decoupled | High scale, eventual consistency OK |
| Client-Server | Clients request, servers provide. 2/3/N-tier variants | Standard web apps |
| Layered (N-Tier) | Horizontal layers: Presentation → Business → Persistence → DB | Enterprise CRUD, team org |
| Publish-Subscribe | Publishers → topics ← subscribers. 1-to-many broadcast | Notifications, cross-service events |
| Peer-to-Peer | Decentralized, each node = client+server. DHT, gossip | Blockchain, file sharing, IoT |
| Monolithic | Single deployable unit. Modular monolith = internal boundaries | Early-stage, simpler ops |
| Microservices | Small independent services, DB per service, API Gateway, service discovery | Large teams, independent scaling |
| Distributed | CAP theorem (pick 2 of C/A/P). PACELC extension. 8 Fallacies | Global systems, HA required |

## Architectural Patterns
- **MVC**: Model (data+logic), View (presentation), Controller (input routing). "Fat Model, Skinny Controller." SPA: View → client-side.
- **Microkernel**: Core (minimal) + Plugins (extensions). Plugin registry. IDE/OS/extensible systems. Go: iface-based registry.
- **Blackboard**: Shared workspace + Knowledge Sources + Control Shell. Non-deterministic problems. AI/speech/signal processing.
- **Serverless**: FaaS, event-triggered, ephemeral, pay-per-use. Cold starts. State must be externalized. Go: fast cold starts.
- **Layered (pattern)**: Closed layers = isolation; Open layers = bypass (anti-sinkhole). Dependencies point down (vs. Hexagonal: inward).

## Enterprise Patterns
- **DDD Tactical**: Entity (identity, mutable, lifecycle), Value Object (no identity, immutable, self-validating), Aggregate Root (consistency boundary), Repository (collection-like, one per aggregate root). Interface in Domain, impl in Infrastructure.
- **DDD Strategic**: Bounded Context (model boundary), Ubiquitous Language (shared terminology), Context Map (ACL, partnership). Aligns with microservice boundaries.
- **Rich vs Anemic Model**: Rich = logic in domain objects. Anemic = data bags + service logic. Rich for complex business rules.
- **DTOs & Mappers**: DTO = pure data container (decouple API from domain). Mapper = transform between layers. Go: manual mapping preferred.
- **CQRS**: Separate read (Query) and write (Command) models. Async projections. Eventually consistent. Use when read/write load asymmetry.
- **Event Sourcing**: State = append-only event log. Replay to reconstruct. Snapshots for perf. Benefits: audit trail, time travel, debug replay.
- **Identity Map**: Cache loaded entities per session (PK → object). Ensures object identity (`a == b`). ORM persistence context.

## Design Rules
1. Don't depend on things you might not need. (YAGNI)
2. Dependencies point inward toward policy, never outward toward detail.
3. One module, one reason to change. (SRP)
4. Code against contracts, not implementations.
5. Make the change easy, then make the easy change.
6. If you need a comment to explain what code does, rename it.
7. Never pass null when you can pass a Null Object or empty collection.
8. Small, focused interfaces > large, bloated ones.
9. Don't build a distributed monolith — share nothing at the database level.
10. Leave the code better than you found it. (Boy Scout Rule)
