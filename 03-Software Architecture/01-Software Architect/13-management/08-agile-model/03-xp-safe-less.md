---
---

## Summary
While Scrum and Kanban handle team-level agility, large organizations need frameworks to scale these practices. This note covers **XP (Extreme Programming)**, which focuses on technical excellence, and scaling frameworks like **SAFe (Scaled Agile Framework)** and **LeSS (Large-Scale Scrum)**, which organize multiple teams around a shared architecture.

## 1. XP (Extreme Programming)
XP is the most "architecturally aware" agile framework because it mandates specific engineering practices.

*   **Core Practices for Architects**:
    *   **Test-Driven Development (TDD)**: Ensures the architecture is testable by design.
    *   **Pair Programming**: Real-time code review and knowledge sharing.
    *   **Continuous Integration**: The system is integrated and tested multiple times a day.
    *   **Simple Design (YAGNI)**: "You Ain't Gonna Need It". Architects should avoid over-engineering.
    *   **Refactoring**: The architecture is not fixed; it evolves.

## 2. SAFe (Scaled Agile Framework)
SAFe is a heavy, prescriptive framework for large enterprises. It explicitly defines the role of the architect.

*   **Architect Roles**:
    *   **System Architect**: Defines the architecture for an Agile Release Train (ART) - a group of 5-12 teams.
    *   **Solution Architect**: Defines the architecture for large solutions requiring multiple ARTs.
    *   **Enterprise Architect**: Aligns strategy with technology across the portfolio.
*   **Key Concept: The Architectural Runway**:
    *   Code, components, and technical infrastructure needed to implement near-term features without excessive delay.
    *   Architects are responsible for laying this track *ahead* of the feature trains.

## 3. LeSS (Large-Scale Scrum)
LeSS is a lightweight framework that applies Scrum to multiple teams working on a *single* product.

*   **Philosophy**: "More with LeSS". Avoids adding extra roles (like the SAFe "System Architect") if possible.
*   **Architectural Approach**:
    *   **One Product Backlog**: All teams work from a single prioritized list.
    *   **Travelers**: Senior architects "travel" between teams to mentor and ensure consistency, rather than sitting in an ivory tower.
    *   **Component Teams vs. Feature Teams**: LeSS strongly favors Feature Teams (end-to-end delivery) over Component Teams, which requires a decoupled architecture.

## Comparison Table

| Feature | XP | SAFe | LeSS |
| :--- | :--- | :--- | :--- |
| **Focus** | Engineering Practices | Enterprise Coordination | Scaling Scrum Simplicity |
| **Architect Role** | Embedded / Everyone | Explicit (System/Solution Arch) | Mentor / Traveler |
| **Key Artifact** | Unit Tests, Simple Code | Architectural Runway | Single Product Backlog |
| **Best For** | Improving Code Quality | Huge Corps / Regulated Industries | 2-8 Teams / Product Corps |

## Application in Go (Refactoring/XP)

XP emphasizes **Refactoring** as a daily habit. In Go, this often means checking for code smells.

```go
// BEFORE: Complex, hard to test (Violation of Simple Design)
func ProcessUserData(data string) {
    // 50 lines of parsing logic
    // 20 lines of validation
    // 30 lines of database calls
}

// AFTER: Refactored (XP Style)
// - Extract Method
// - Single Responsibility
// - Testable

func ProcessUserData(data string) error {
    u, err := parseUser(data) // Pure function, easy to unit test
    if err != nil {
        return err
    }
    return saveUser(u) // Interface-based dependency
}
```

## Interview Questions

**Q: In SAFe, what is the "Architectural Runway"?**
**A:** It consists of the existing code, components, and technical infrastructure necessary to support the implementation of prioritized features. If the runway is too short, the train (development teams) will run out of track (cannot deliver features due to technical blockers).

**Q: How does XP differ from Scrum regarding engineering practices?**
**A:** Scrum is a management framework; it doesn't tell you *how* to write code. XP is an engineering framework; it mandates practices like TDD, Pair Programming, and CI. Many successful teams use "Scrum with XP engineering practices."

**Q: Why does LeSS prefer "Feature Teams" over "Component Teams" and how does that affect architecture?**
**A:** Component teams (e.g., "The DB Team", "The UI Team") create silos and handoffs. Feature teams deliver end-to-end value. To support Feature Teams, the architecture must be decoupled so that Team A can touch the UI, API, and DB without breaking Team B's work (e.g., via Microservices or well-structured Monoliths).
