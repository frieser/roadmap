---
---

## Summary
A Software Architect acts as a "Force Multiplier" for the engineering team. Consulting and coaching involves mentoring developers, unblocking complex technical issues, and evangelizing best practices. Unlike a manager who focuses on career pathing, the architect focuses on technical excellence and empowering developers to make sound architectural decisions themselves.

## Detailed Explanation

### Mentorship vs. Coaching vs. Consulting
*   **Mentorship**: Long-term relationship focused on the developer's technical growth and broad architectural thinking.
*   **Coaching**: Short-term, task-specific guidance (e.g., helping a developer implement a specific design pattern).
*   **Consulting**: Providing expert advice on a specific problem or reviewing a design proposal.

### Key Coaching Activities

#### 1. Targeted Code Reviews
Architects shouldn't review every PR. Instead, they focus on:
*   Critical paths (authentication, database migrations).
*   Integration points.
*   Compliance with defined architectural patterns.
*   **Goal**: Turn the review into a teaching moment, explaining *why* a different approach might be better.

#### 2. Architecture Workshops / "Brown Bag" Sessions
Regularly hosting sessions to teach new concepts (e.g., "Intro to Event Sourcing" or "How to use Go Channels effectively").

#### 3. Pair Programming on "Hard Problems"
When a team is stuck on a complex bug or a difficult refactor, the architect joins them in the trenches. This builds trust and provides high-bandwidth knowledge transfer.

### Go Application: Idiomatic Code Guidance
Coaching in a Go environment often involves guiding developers away from "Java-isms" or "Python-isms" and towards **Idiomatic Go**.

**Example: Coaching on Error Handling**
A developer might try to use `panic/recover` or return broad `error` types. The architect coaches them on wrapping errors and using specific error types for better observability.

```go
// Architect's advice: Instead of returning just 'err', 
// wrap it with context to make debugging easier.

import "fmt"

func (s *Service) ProcessUser(id string) error {
    user, err := s.repo.FindByID(id)
    if err != nil {
        // Coaching point: fmt.Errorf with %w enables error wrapping (Go 1.13+)
        return fmt.Errorf("failed to process user %s: %w", id, err)
    }
    return nil
}
```

### Measuring Success
The success of an architect's coaching is measured by:
*   **Team Autonomy**: Developers start making correct architectural decisions without intervention.
*   **Code Quality**: A measurable decrease in "architecture-smell" bugs.
*   **Knowledge Diffusion**: Senior developers start coaching juniors using the same patterns the architect taught them.

## Interview Questions

**Q: How do you balance being "hands-on" with your architectural responsibilities?**
**A:** I aim for a "Lead by Example" approach. I stay hands-on through high-impact activities like pairing on complex features, building shared libraries, or developing "Skeleton Projects." This ensures I stay grounded in the reality of the codebase while still fulfilling my high-level duties.

**Q: A senior developer strongly disagrees with an architectural pattern you've introduced. How do you handle this?**
**A:** I treat it as a technical discussion, not a hierarchy issue. I listen to their concerns—they might have valid local context I missed. We compare the trade-offs of both approaches. If we can't agree, I might suggest a small-scale POC (Proof of Concept) to test both patterns in production before making a final call.

**Q: How do you identify which developers need coaching and what topics to cover?**
**A:** I look for patterns in code reviews and post-mortems. If I see multiple teams struggling with "Leaky Abstractions" or "Race Conditions," I organize a workshop. I also encourage developers to self-identify their "unknown unknowns" during informal 1-on-1s.
