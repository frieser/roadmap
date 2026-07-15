---
---

## Comparison

| Diagram | Purpose | Key Elements |
|---------|---------|--------------|
| **Class** | Static structure, code blueprint | Classes, inheritance, composition, interfaces |
| **Use Case** | Functional requirements, scope | Actors, ovals, `<<include>>`, `<<extend>>` |
| **Activity** | Control flow + concurrency | Fork/join, decisions, activities |
| **State Machine** | Single object lifecycle | States, transitions, events, guards |
| **Sequence** | Object interactions over time | Lifelines, sync/async messages |

## Class Relationships (most → least coupled)

| Relationship | Arrow | Go |
|-------------|-------|-----|
| Inheritance | Solid hollow triangle | Struct embedding |
| Realization | Dashed hollow triangle | Interface |
| Composition | Solid filled diamond | Struct value field |
| Aggregation | Solid hollow diamond | Slice of pointers |
| Dependency | Dashed arrow | Parameter/import |

## Decision Matrix

| Need to model… | Diagram |
|----------------|---------|
| Code structure | Class |
| User goals | Use Case |
| Algorithm flow | Activity |
| Object lifecycle | State Machine |
| API interactions | Sequence |

## Cross-Diagram Links

- Use Case → Activity/Sequence: one scenario per diagram
- Class → State Machine: one class = one state diagram
- Class → Sequence: classes = lifelines
