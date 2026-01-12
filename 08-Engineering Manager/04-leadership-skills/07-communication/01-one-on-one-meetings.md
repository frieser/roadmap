## Summary
One-on-one (1:1) meetings are the most critical tool in an engineering manager's arsenal. They are the only regular time dedicated solely to the direct report. The primary purpose is relationship building, career development, and unblocking issues—**not** status updates. It is the report's meeting, not the manager's.

## Detailed Explanation
### The Golden Rule
**"The 1:1 belongs to the employee."** They set the agenda. If they don't have one, the manager should have a fallback list of open-ended questions.

### Key Topics
*   **Well-being**: "How are you feeling about work/life balance?"
*   **Career Growth**: "What skills do you want to learn next quarter?"
*   **Feedback**: Mutual feedback (Manager <-> Report).
*   **Blockers**: "Where are you stuck?"

### Anti-Patterns
*   Canceling frequently (signals "you are not important").
*   Turning it into a status report (use Jira/Standup for that).
*   Doing all the talking.

### Go Code Example: The 1:1 Structure
This code models a 1:1 agenda, ensuring the employee's topics take precedence over the manager's.

```go
package communication

import "fmt"

type Topic struct {
	Title    string
	Owner    string // "Employee" or "Manager"
	Category string // "Career", "Project", "Personal"
}

type OneOnOne struct {
	EmployeeName string
	Duration     int // minutes
	Topics       []Topic
}

// Conduct runs the meeting simulation
func (m *OneOnOne) Conduct() {
	fmt.Printf("Starting 1:1 with %s (%d mins)\n", m.EmployeeName, m.Duration)
	
	// Filter and prioritize Employee topics
	fmt.Println("--- Employee Agenda (Priority) ---")
	for _, t := range m.Topics {
		if t.Owner == "Employee" {
			fmt.Printf("- [%s] %s\n", t.Category, t.Title)
		}
	}

	// Manager topics only if time permits
	fmt.Println("--- Manager Agenda (If time permits) ---")
	for _, t := range m.Topics {
		if t.Owner == "Manager" {
			fmt.Printf("- [%s] %s\n", t.Category, t.Title)
		}
	}
}

func main() {
	meeting := OneOnOne{
		EmployeeName: "Sarah",
		Duration:     30,
		Topics: []Topic{
			{"Discuss promotion path", "Employee", "Career"},
			{"Frustrated with CI pipeline", "Employee", "Blocker"},
			{"Review Q3 Goals", "Manager", "Planning"}, // This comes last
		},
	}
	
	meeting.Conduct()
}
```

## Interview Questions
**Q: How do you structure your 1:1s?**
**A:** I schedule them weekly for 30-45 minutes. I use a shared document for the agenda. I start by asking "How are you?" to check well-being. Then we cover their agenda items. I reserve the last 10 minutes for my items and coaching. I explicitly ban status updates unless they are blocked.

**Q: What do you do if an employee is silent during 1:1s?**
**A:** I don't fill the silence immediately. I might ask open-ended questions like "What part of your day do you look forward to most?" or "If you could change one thing about the team, what would it be?" If silence persists, I might take a walk-and-talk (if in person) or switch to a less formal setting to reduce pressure.

**Q: How do you handle a 1:1 when you have nothing to discuss?**
**A:** I never cancel. "Nothing to discuss" is often a signal of disengagement or burnout. I use that time to connect personally, discuss long-term career goals, or do a "health check" on our working relationship. "Since we have time, let's look at where you want to be in 2 years."
