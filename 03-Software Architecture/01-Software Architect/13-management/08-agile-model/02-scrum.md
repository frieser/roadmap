---
---

## Summary
**Scrum** is an agile framework for developing, delivering, and sustaining complex products. It is an iterative and incremental approach that emphasizes collaboration, accountability, and iterative progress. For a Software Architect, Scrum provides the rhythm (Sprints) for delivering architectural value and the ceremonies to align with the team.

## Detailed Explanation

Scrum is built on **Empiricism** (Transparency, Inspection, Adaptation) and involves specific Roles, Events, and Artifacts.

### 1. The Roles
*   **Product Owner (PO)**: Holds the vision. Prioritizes the Product Backlog. Architects must collaborate with the PO to prioritize "Technical Enablers" alongside business features.
*   **Scrum Master**: Servant-leader who removes impediments. Helps the architect if technical blockers (like CI/CD issues) are slowing down the team.
*   **Developers**: The cross-functional team (including QA, UX, and Architects) who create the Increment.

### 2. The Events (Ceremonies)
*   **Sprint Planning**: Where the team selects work. The architect ensures the team selects tasks that are architecturally feasible and ready (Definition of Ready).
*   **Daily Scrum**: 15-minute sync. Architects attend to hear about technical blockers.
*   **Sprint Review**: Demoing the increment. Architects demonstrate technical wins (e.g., "We reduced API latency by 50%").
*   **Sprint Retrospective**: Improving the process. Architects use this to discuss code quality, debt, and tooling issues.

### 3. The Artifacts
*   **Product Backlog**: The master list. Architects should inject "Architectural Debt" and "Spike" items here.
*   **Sprint Backlog**: The plan for the current Sprint.
*   **Increment**: The potentially skippable product increment.

### Architectural Challenges in Scrum
*   **"No time for design"**: Scrum moves fast. Architects must work one or two sprints *ahead* of the team (often called "Sprint 0" behavior or "Architecture Runway") to prepare designs before developers pick up the tickets.
*   **The "Definition of Done" (DoD)**: Architects are responsible for ensuring the DoD includes technical standards like "Unit Tests passed," "Code Reviewed," and "No new SonarQube critical issues."

## Mermaid Diagram: Scrum Cycle

```mermaid
graph LR
    Backlog[Product Backlog] --> Plan[Sprint Planning]
    Plan --> Sprint[Sprint (1-4 Weeks)]
    Sprint --> Daily((Daily Scrum))
    Sprint --> Review[Sprint Review]
    Sprint --> Retro[Sprint Retro]
    Review --> Increment[Product Increment]
    Retro --> Plan
```

## Go Application Context

In a Scrum team using Go, the "Definition of Done" often translates to CI/CD pipeline checks.

```go
// Definition of Done Check: "All public functions must have documentation."
// This is enforced by linters like `golint` or `revive` during the Sprint.

// Good: Compliant with DoD
// ProcessPayment handles the transaction logic.
func ProcessPayment(amount int) error {
	return nil
}

// Bad: Fails DoD (No comment)
func ProcessRefund(amount int) error {
	return nil
}
```

## Interview Questions

**Q: How does a Software Architect fit into a Scrum Team?**
**A:** The architect can either be a member of the Scrum Team (working on complex tasks) or an advisor to multiple teams. Crucially, the architect works with the Product Owner to ensure the **Product Backlog** contains necessary architectural work (Refactoring, Upgrades) and ensures the **Definition of Done** maintains high quality.

**Q: What is "Sprint 0" and is it part of official Scrum?**
**A:** "Sprint 0" is a controversial term for the initial setup phase (setting up environments, initial architecture) before regular sprints begin. It is **not** part of official Scrum (which says every sprint must produce value), but many architects use it to lay the "Walking Skeleton" of the architecture.

**Q: How do you handle a major architectural refactor in Scrum?**
**A:** You don't pause the project for months. You break the refactor down into small, valuable slices that can be delivered in single Sprints (Strangler Fig Pattern). You prioritize these slices in the Product Backlog with the PO.
