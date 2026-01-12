## Summary
Stakeholder management is the process of identifying individuals who have an interest in your project and managing their expectations and influence. Success is often defined not just by the code working, but by stakeholders *feeling* that their needs were met. It involves constant communication, negotiation, and alignment.

## Detailed Explanation
### The Power/Interest Grid
Map stakeholders to determine your strategy:
1.  **High Power, High Interest**: Manage Closely. (Key decision makers).
2.  **High Power, Low Interest**: Keep Satisfied. (Meet their needs, don't bore them).
3.  **Low Power, High Interest**: Keep Informed. (They can be allies or noise).
4.  **Low Power, Low Interest**: Monitor. (Minimum effort).

### Key Strategies
*   **Early Alignment**: Agree on success metrics before writing code.
*   **No Surprises**: Bad news must be delivered personally and early.
*   **Speak Their Language**: Talk revenue to Sales, reliability to Ops.

### Go Code Example: Stakeholder Mapping
This code helps classify stakeholders and suggests a communication strategy.

```go
package leadership

import "fmt"

type Stakeholder struct {
	Name     string
	Power    int // 1-10
	Interest int // 1-10
}

// GetStrategy returns the engagement model based on the Power/Interest Grid
func (s *Stakeholder) GetStrategy() string {
	if s.Power > 5 && s.Interest > 5 {
		return "Manage Closely (Weekly Syncs, Direct Input)"
	}
	if s.Power > 5 && s.Interest <= 5 {
		return "Keep Satisfied (High-level updates, Avoid blockers)"
	}
	if s.Power <= 5 && s.Interest > 5 {
		return "Keep Informed (Newsletters, Demos)"
	}
	return "Monitor (Ad-hoc updates)"
}

func main() {
	stakeholders := []Stakeholder{
		{"CTO", 10, 10},          // High Power, High Interest
		{"Marketing VP", 8, 3},   // High Power, Low Interest
		{"Junior Dev", 2, 9},     // Low Power, High Interest
	}

	for _, sh := range stakeholders {
		fmt.Printf("Stakeholder: %-15s | Strategy: %s\n", sh.Name, sh.GetStrategy())
	}
}
```

## Interview Questions
**Q: How do you manage a stakeholder who keeps changing requirements?**
**A:** I use a "Change Request" process. I don't say "no," but I show the *cost* of the change. "We can add that feature, but it will push the release date by 3 days. Do you want to swap it with something else to keep the date?" This puts the trade-off decision back on them.

**Q: How do you handle a disagreement between two powerful stakeholders?**
**A:** I bring them into the same room (or call). I act as a facilitator, clarifying the conflicting requirements and the impact on the engineering team. I try to find a common business goal they share. If they can't agree, I escalate to their common boss or the sponsor, explaining that the team is blocked until a decision is made.

**Q: How do you say 'No' to a stakeholder?**
**A:** I use the "Yes, and..." or "No, because..." technique. "Yes, we can do that, and it will require dropping feature Y." or "We can't do that right now because our focus is on stability for Black Friday, but we can put it in the Q1 backlog." I validate their request but explain the constraints.
