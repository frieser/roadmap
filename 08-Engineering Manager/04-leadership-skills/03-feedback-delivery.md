## Summary
Feedback delivery is the skill of providing information to individuals about their performance or behavior in a way that leads to positive change. Effective feedback reinforces good behavior (positive feedback) and redirects unhelpful behavior (constructive feedback). It is essential for growth and is most effective when it is specific, timely, and actionable.

## Detailed Explanation
### The SBI Model
The standard for clear feedback:
*   **Situation**: When and where did it happen? (Be specific).
*   **Behavior**: What exactly did they do? (Observable actions, not assumptions).
*   **Impact**: What was the result? (On the team, product, or you).

### Radical Candor
A framework by Kim Scott suggesting that to be a good boss, you must:
1.  **Care Personally**: Build a real relationship.
2.  **Challenge Directly**: Be willing to give the tough news.
High Care + High Challenge = Radical Candor.

### Go Code Example: The Feedback Struct
This struct enforces the SBI model, preventing "vague" feedback which is often rejected or misunderstood.

```go
package leadership

import (
	"errors"
	"fmt"
)

// Feedback represents a structured piece of feedback using the SBI model.
type Feedback struct {
	Situation string // Context
	Behavior  string // Observable action
	Impact    string // Result
}

// Deliver validates and sends the feedback.
func (f *Feedback) Deliver(recipient string) error {
	// Validation: Feedback must be specific
	if f.Situation == "" || f.Behavior == "" || f.Impact == "" {
		return errors.New("feedback is vague: must include Situation, Behavior, and Impact")
	}

	// Simulation of delivery
	fmt.Printf("--- Feedback for %s ---\n", recipient)
	fmt.Printf("When you... %s\n", f.Situation)
	fmt.Printf("You... %s\n", f.Behavior)
	fmt.Printf("The result was... %s\n", f.Impact)
	fmt.Println("-----------------------")
	
	return nil
}

func main() {
	// BAD Feedback example
	badFeedback := Feedback{Behavior: "You were rude."}
	if err := badFeedback.Deliver("Alice"); err != nil {
		fmt.Println("Failed to deliver:", err)
	}

	// GOOD Feedback example (SBI)
	goodFeedback := Feedback{
		Situation: "During the incident response meeting yesterday,",
		Behavior:  "you interrupted the junior engineer three times while they were explaining the logs,",
		Impact:    "which caused them to shut down and we missed critical information they had found.",
	}
	
	goodFeedback.Deliver("Bob")
}
```

## Interview Questions
**Q: How do you deliver negative feedback to a high performer?**
**A:** High performers usually crave growth, so I frame it as "what's next." I acknowledge their high standard but point out the specific area (e.g., soft skills, mentorship) holding them back from the *next* level. "You crushed the code, but by not documenting it, you've made yourself a bottleneck. To get to Staff Engineer, you need to scale yourself."

**Q: Describe a time you received difficult feedback. How did you handle it?**
**A:** My manager told me I was shielding the team too much, preventing them from understanding the business pressure. Initially, I was defensive because I thought I was "protecting" them. However, I reflected on the *Impact* (team lacked context), accepted the feedback, and started sharing more raw business metrics in our sprint planning.

**Q: Why is timely feedback important?**
**A:** Feedback has a half-life. If I tell you about a mistake you made three weeks ago, you won't remember the context, and it feels like I've been "saving it up" to use against you. Immediate feedback allows for immediate correction and shows that I'm paying attention to help you, not to judge you.
