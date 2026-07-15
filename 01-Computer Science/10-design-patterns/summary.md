---
---

# Design Patterns — Ultra-Compact Summary

## GoF 23 — Single Table
| Name | Category | 5-Word Purpose | Go Fit |
|------|----------|---------------|--------|
| Singleton | Creational | One instance, global access point | rarely |
| Factory Method | Creational | Subclasses decide which object created | yes |
| Abstract Factory | Creational | Create families of related objects | yes |
| Builder | Creational | Separate construction from object representation | yes |
| Prototype | Creational | Clone existing object for creation | rarely |
| Adapter | Structural | Make incompatible interfaces work together | yes |
| Bridge | Structural | Separate abstraction from implementation details | yes |
| Composite | Structural | Treat individual and composite uniformly | yes |
| Decorator | Structural | Add behavior without multiple subclasses | yes |
| Facade | Structural | Single interface to complex subsystem | yes |
| Flyweight | Structural | Share common state across objects | yes |
| Proxy | Structural | Surrogate controls access to object | yes |
| Chain of Resp. | Behavioral | Pass request along handler chain | yes |
| Command | Behavioral | Encapsulate request as command object | yes |
| Interpreter | Behavioral | Parse and evaluate language sentences | rarely |
| Iterator | Behavioral | Access elements without exposing internals | yes |
| Mediator | Behavioral | Central hub for object communication | yes |
| Memento | Behavioral | Save and restore previous state | rarely |
| Observer | Behavioral | Notify dependents of state changes | yes |
| State | Behavioral | Alter behavior by internal state | yes |
| Strategy | Behavioral | Swap interchangeable algorithm implementations | yes |
| Template Method | Behavioral | Define skeleton, defer steps downstream | no |
| Visitor | Behavioral | Add operations without modifying classes | rarely |

## Go Built-in Pattern Replacements
- `range` → Iterator. `chan` → Observer + Mediator. funcs → Strategy + Command.
- Embedding + Interfaces → Proxy + Decorator. `sync.Once` → Singleton (last resort).

## Idiomatic Go: Functional Options (Builder)
- `type Option func(*Server)`. `NewServer(WithPort(9000), WithTLS())`. Chained config.

## Architectural Patterns
| Pattern | Core | Go Fit |
|---------|------|--------|
| Layered (N-Tier) | Presentation → Business → Data → DB | Legacy |
| Clean/Hexagonal | Deps point inward. Core defines ports. | Dominant |
| Microservices | Loose services via HTTP/gRPC | Go excels |
| Event-Driven | Async broker (Kafka). Emit + consume events. | Channels + broker |

## Hexagonal Structure (Go Idiom)
- `cmd/api/main.go` — wiring.
- `internal/core/domain/` — entities (zero deps).
- `internal/core/ports/` — interfaces (Ports).
- `internal/service/` — use cases (depends on ports only).
- `internal/adapters/handler/` — HTTP (driving adapter).
- `internal/adapters/repository/` — DB (driven adapter, implements port).
- Rule: Source deps point inward. adapters → service → domain. NEVER reversed.

## Dependency Injection
- Constructor injection: `NewService(repo Repo) *Service`. Idiomatic Go.
- Wire (Google): Compile-time codegen. Fx (Uber): Runtime reflection. Manual: `main.go` wiring.
- Benefit: Interfaces enable mocks. No global state. Explicit dependency graph.

## Null Object Pattern
- No-op interface impl. Eliminates nil checks. Constructor guard: `if l == nil { l = &NullLogger{} }`.
- Risk: Silent failure — missing dep → NullObject absorbs → bug hidden.

## Type Object Pattern
- Entity (Monster) holds ptr to Type (MonsterType). Types loadable from config at runtime.
- Avoids subclass explosion. Type Objects as Flyweights: one shared instance per type.
- Type Object = shared data. Strategy = shared behavior. Distinct patterns.

## Cross-Pattern Decision (Go)
| Need | Pattern |
|------|---------|
| Flexible creation | Factory Method / Functional Options |
| Family of objects | Abstract Factory |
| Many config params | Functional Options (Builder) |
| Incompatible interfaces | Adapter |
| Add behavior w/o mod | Decorator (middleware) |
| Simplify subsystem | Facade |
| Tree structures | Composite |
| Pipeline processing | Chain of Responsibility |
| Undo/redo | Command or Memento |
| State-dependent | State |
| Pluggable algorithms | Strategy |
| Broadcast changes | Observer (channels) |
| New Go backend | Clean/Hexagonal |
| Indie team deploys | Microservices |
| Async decoupling | Event-Driven |

## Key Interview Answers (Fragments)
- Singleton discouraged: Global state. Untestable. DI preferred.
- Go Decorator: Interface wrapping. `http.Handler` middleware.
- Hexagonal > Layered: Dep inversion — logic defines interface, DB implements it.
- Constructor > Setter: Guarantees full initialization. No nil fields.
- Monolith first: Go packages + `internal/` excel at large structured monoliths.
- Null Object risk: Hides config errors. Silent no-op instead of alert.
- Type Object vs Strategy: Type Object = shared data. Strategy = shared behavior.
- `internal/` role: Compiler-enforced encapsulation. Not importable externally.
