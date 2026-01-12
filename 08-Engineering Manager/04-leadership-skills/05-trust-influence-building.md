## Summary
Trust is the currency of leadership. Without trust, influence is impossible, and management becomes micromanagement. Building trust requires a combination of character (integrity, intent) and competence (capability, results). Influence is the ability to affect others' actions and decisions without having direct authority over them.

## Detailed Explanation
### The Trust Equation
Trust = (Credibility + Reliability + Intimacy) / Self-Orientation

*   **Credibility**: "I know what I'm talking about." (Technical competence).
*   **Reliability**: "I do what I say I will do." (Consistency).
*   **Intimacy**: "I feel safe sharing with you." (Empathy/Discretion).
*   **Self-Orientation**: "I focus on me vs. we." (High self-orientation reduces trust).

### Influence Strategies
*   **Reciprocity**: Help others, and they will help you.
*   **Social Proof**: "Team X is doing it this way."
*   **Expertise**: Leading with data and knowledge.

### Go Code Example: The Trust Meter
This code models how a manager's actions affect their trust score with a report, based on the Trust Equation.

```go
package leadership

import "fmt"

type Manager struct {
	Name            string
	Reliability     float64 // 0-10
	Credibility     float64 // 0-10
	Intimacy        float64 // 0-10
	SelfOrientation float64 // 1-10 (Lower is better)
}

// TrustScore calculates the Trust Equation
func (m *Manager) TrustScore() float64 {
	numerator := m.Credibility + m.Reliability + m.Intimacy
	if m.SelfOrientation < 1 {
		m.SelfOrientation = 1 // Prevent division by zero
	}
	return numerator / m.SelfOrientation
}

// Action represents a management behavior
func (m *Manager) PerformAction(action string) {
	switch action {
	case "Missed 1:1":
		fmt.Println("Action: Missed 1:1 -> Reliability drops.")
		m.Reliability -= 2
	case "Admitted Mistake":
		fmt.Println("Action: Admitted Mistake -> Intimacy up, Self-Orientation down.")
		m.Intimacy += 2
		m.SelfOrientation -= 1
	case "Solved Hard Bug":
		fmt.Println("Action: Solved Hard Bug -> Credibility up.")
		m.Credibility += 1
	}
}

func main() {
	mgr := Manager{
		Name:            "David",
		Reliability:     5,
		Credibility:     8,
		Intimacy:        4,
		SelfOrientation: 5,
	}

	fmt.Printf("Initial Trust Score: %.2f\n", mgr.TrustScore())
	
	mgr.PerformAction("Admitted Mistake")
	
	fmt.Printf("New Trust Score: %.2f\n", mgr.TrustScore())
}
```

## Interview Questions
**Q: How do you build trust with a new team?**
**A:** I start by listening. For the first 30 days, I don't make big changes; I focus on understanding their context, pain points, and history (Intimacy). I look for "quick wins"—small annoyances I can fix for them (e.g., getting them better licenses or cancelling a useless meeting) to demonstrate I am there to serve them (Reliability). I also admit what I don't know (Vulnerability).

**Q: How do you influence a stakeholder who disagrees with you?**
**A:** I try to understand their underlying motivation (Self-Orientation vs Company Goal). I speak their language—if they are Product, I talk about user value; if Finance, I talk about cost. I use data to support my argument rather than opinion, and if possible, I run a small experiment (POC) to prove the concept before asking for a full commitment.

**Q: What is the most important factor in the Trust Equation?**
**A:** Self-Orientation. No matter how smart (Credibility) or reliable you are, if people sense you are only in it for your own promotion or glory, they will not trust you. Lowering self-orientation—truly caring about the team's success over your own—is the fastest way to build deep trust.
