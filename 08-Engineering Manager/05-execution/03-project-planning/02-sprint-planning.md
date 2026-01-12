## Summary
Sprint Planning is a core Scrum ceremony where the team determines what work can be performed in the upcoming Sprint and how that work will be achieved. The outcome is a defined Sprint Goal and a Sprint Backlog.

## Detailed Explanation
Sprint Planning aligns the Product Owner's priorities with the Engineering Team's capacity.

### The Process
1.  **Product Owner** presents the highest priority items from the Product Backlog.
2.  **The Team** asks clarifying questions to understand acceptance criteria.
3.  **The Team** estimates the effort (if not done in refinement).
4.  **The Team** pulls items into the Sprint Backlog based on their historical velocity.
5.  **The Team** breaks stories down into technical tasks (optional but recommended).
6.  **Consensus**: The team commits to the Sprint Goal.

### Key Metrics
*   **Velocity**: The average number of story points the team completes per sprint. Used to forecast capacity.
*   **Capacity**: Total available man-hours/days for the specific sprint (minus holidays/leave).

## Go Code Example
Simulating a Sprint Planning session where items are pulled from the backlog until velocity capacity is reached.

```go
package main

import (
	"fmt"
)

type Story struct {
	Title  string
	Points int
}

type Sprint struct {
	Capacity      int
	CurrentPoints int
	Backlog       []Story
}

// Plan attempts to fit stories into the sprint based on capacity
func (s *Sprint) Plan(productBacklog []Story) []Story {
	var remainingBacklog []Story
	
	for _, story := range productBacklog {
		if s.CurrentPoints+story.Points <= s.Capacity {
			s.Backlog = append(s.Backlog, story)
			s.CurrentPoints += story.Points
			fmt.Printf("Accepted: %s (%d pts)\n", story.Title, story.Points)
		} else {
			fmt.Printf("Skipped: %s (%d pts) - Exceeds capacity\n", story.Title, story.Points)
			remainingBacklog = append(remainingBacklog, story)
		}
	}
	return remainingBacklog
}

func main() {
	// Historical Velocity = 20
	sprint := Sprint{Capacity: 20, CurrentPoints: 0}

	productBacklog := []Story{
		{"User Login", 5},
		{"Password Reset", 3},
		{"Data Export", 8},
		{"Admin Dashboard", 13}, // Too big?
		{"Fix Typo", 1},
	}

	fmt.Println("--- Starting Sprint Planning ---")
	leftovers := sprint.Plan(productBacklog)

	fmt.Printf("\nSprint Committed: %d/%d points\n", sprint.CurrentPoints, sprint.Capacity)
	fmt.Printf("Remaining in Backlog: %d items\n", len(leftovers))
}
```

## Interview Questions
**Q: What happens if you finish all sprint items early?**
**A:** We consult the Product Owner to pull in the next highest priority item from the backlog that fits the remaining time. Alternatively, we use the time for technical debt, learning, or helping other team members. We do *not* just stop working.

**Q: Who decides how many points go into a sprint?**
**A:** The Development Team. The Product Owner decides *priority*, but only the team can decide *how much* work they can commit to. Management cannot dictate velocity.

**Q: How long should Sprint Planning take?**
**A:** Time-boxed to 2 hours per week of sprint duration (e.g., 4 hours for a 2-week sprint). If it takes longer, it indicates the backlog was not properly refined beforehand.
