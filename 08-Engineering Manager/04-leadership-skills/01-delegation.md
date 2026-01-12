## Summary
Delegation is the act of assigning authority and responsibility to others to carry out specific activities. Effective delegation is not just "dumping" work; it involves matching tasks to the right people, defining clear outcomes (the "what," not the "how"), and providing the necessary support and autonomy. It is crucial for scaling a manager's impact and developing the team's skills.

## Detailed Explanation
### The Levels of Delegation
1.  **Wait to be told**: "Do exactly what I say." (Low trust/skill).
2.  **Ask what to do**: "Here is the problem, what should I do?"
3.  **Recommend**: "I recommend we do X." (Manager approves).
4.  **Act and report**: "I did X." (High trust).
5.  **Act**: "Handle it entirely." (Full autonomy).

### Key Principles
*   **Delegate outcomes, not methods**: Let them figure out the "how."
*   **Match skill and challenge**: Ensure the task stretches them but doesn't break them (Flow state).
*   **Provide context**: Explain *why* this task matters.

### Go Code Example: The Delegation Logic
This code models a manager's decision-making process when delegating a task, checking for capacity and skill alignment.

```go
package leadership

import (
	"errors"
	"fmt"
)

// Task represents a unit of work.
type Task struct {
	Name       string
	Difficulty int // 1-10 scale
	RequiredSkill string
}

// Engineer represents a team member.
type Engineer struct {
	Name     string
	Skills   map[string]int // Skill name -> Proficiency (1-10)
	Capacity int            // Available hours
}

// Manager handles delegation.
type Manager struct {
	Team []Engineer
}

// Delegate attempts to find a suitable engineer for a task.
func (m *Manager) Delegate(t Task) (string, error) {
	for _, eng := range m.Team {
		// Check bandwidth
		if eng.Capacity < 1 {
			continue
		}

		// Check skill alignment
		proficiency, ok := eng.Skills[t.RequiredSkill]
		if !ok {
			continue
		}

		// Delegation Logic:
		// If proficiency is slightly lower than difficulty, it's a "Stretch Goal" (Good for growth)
		// If proficiency is much higher, it might be "Boring" (Bad for motivation)
		if proficiency >= t.Difficulty-2 {
			return fmt.Sprintf("Delegated '%s' to %s (Growth Opportunity)", t.Name, eng.Name), nil
		}
	}

	return "", errors.New("no suitable engineer found for delegation")
}

func main() {
	team := []Engineer{
		{"Alice", map[string]int{"Go": 8}, 10},
		{"Bob", map[string]int{"Go": 4}, 10},
	}
	
	mgr := Manager{Team: team}
	task := Task{"Refactor Core API", 9, "Go"}

	decision, err := mgr.Delegate(task)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println(decision)
	}
}
```

## Interview Questions
**Q: Tell me about a time you delegated a critical task and it went wrong. How did you handle it?**
**A:** I delegated a database migration to a senior engineer but didn't define the rollback criteria clearly. The migration failed, causing downtime. I took responsibility for not reviewing the safety plan (the "what") while respecting their ownership of the script (the "how"). We fixed it together, and I updated our delegation checklist to include mandatory rollback reviews.

**Q: How do you decide what to delegate and what to keep?**
**A:** I use the Eisenhower Matrix. I delegate urgent/important tasks to seniors to unblock me, and non-urgent/important tasks (like research) to juniors for growth. I keep tasks that only I can do (personnel issues, sensitive stakeholder mgmt) or tasks that require my specific authority.

**Q: How do you monitor delegated tasks without micromanaging?**
**A:** I agree on a cadence of updates upfront (e.g., "update me every Tuesday" or "ping me only if blocked"). I focus on the agreed-upon milestones/outcomes rather than checking the daily activity. This establishes trust while ensuring I'm informed.
