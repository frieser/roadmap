---
---

## Summary
A modern Software Architect is not a dictator but a **Consultant** and **Coach**. You cannot force teams to follow your vision; you must influence them. The goal is to elevate the technical capability of the organization so that teams make the right architectural decisions themselves, even when you aren't in the room.

## Detailed Explanation

### The Consultant Role (Strategic)
*   **Influence without Authority**: You often don't manage the developers directly. You must use persuasion, data, and proof-of-concept to convince stakeholders.
*   **Stakeholder Management**: You translate between "Business" (ROI, Time-to-Market) and "Engineering" (Refactoring, Scalability). You are the bridge.
*   **Problem Solver**: Teams call you when they are stuck. You provide options, pros/cons, and guidance, not just orders.

### The Coach Role (Tactical)
*   **Mentoring**: Helping Senior Engineers become Leads or Architects.
*   **Guardrails, not Gates**: Instead of reviewing every line of code (Gatekeeper), build tools, linters, and templates (Guardrails) that make it easy to do the right thing.
*   **Sponsoring**: Giving visibility to others. Let the team present the architecture they designed with your help.

## Go-Specific Context/Examples

Coaching a team moving to Go requires shifting their mindset:
*   **Java/C# -> Go**: Developers often try to write "Java in Go" (factories, deep inheritance). Your job is to coach them on **Idiomatic Go**:
    *   "Accept interfaces, return structs."
    *   "Errors are values, handle them explicitly."
    *   "Keep it simple."
*   **Code Reviews**: Use reviews not just to find bugs, but to teach architectural principles. "Why did we couple these packages? Could we use an interface here to invert the dependency?"

## Interview Questions

**Q: How do you handle a team that disagrees with your architectural decision?**
**A:** First, I listen. They usually know the domain/code better than I do. If their concern is valid, I adapt. If it's a difference of opinion, I explain the "Why" (Strategic Alignment, Long-term cost). If we still disagree, I might ask them to "Disagree and Commit" for a trial period, or if the risk is low, let them try their way and learn from the result.

**Q: What is the difference between a Senior Engineer and an Architect?**
**A:** A Senior Engineer focuses on **"How"** to build a feature best (Clean Code, Implementation). An Architect focuses on **"Where"** that feature belongs (System Design, Boundaries) and **"Why"** we are building it (Trade-offs, Buy vs Build). The Architect's scope is broader (cross-team) and longer-term.

**Q: How do you scale yourself as an architect?**
**A:** By creating documentation (ADRs), paved paths (templates/libraries), and mentoring "Champions" in each team who can advocate for good architecture. I try to make myself redundant for day-to-day decisions.
