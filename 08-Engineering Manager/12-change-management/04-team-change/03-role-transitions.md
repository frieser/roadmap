# Role Transitions

## Summary
Role Transitions occur when an individual moves from IC (Individual Contributor) to Manager, or Junior to Senior. EMs must support these transitions because the skills that got them *here* (coding) won't get them *there* (delegating). The "IC to Manager" pendulum is the most dangerous transition.

## Detailed Explanation

### 1. The Maker vs. Manager Schedule
*   **Maker**: Needs 4-hour blocks of deep work.
*   **Manager**: Lives in 30-minute slots. Context switches constantly.
*   *Transition Pain*: The new manager feels "unproductive" because they didn't write code today.

### 2. Letting Go of the Legos
*   New leaders often micromanage because they trust their own hands more than their team's.
*   *Guidance*: "Your job is not to build the Lego tower. Your job is to ensure the team has the bricks."

### 3. The 90-Day Plan
*   **Day 0-30**: Listen and Learn. Don't change anything.
*   **Day 31-60**: Secure early wins. Fix low-hanging fruit.
*   **Day 61-90**: Set long-term strategy.

## Go Code Example: Time Audit Tool
This concept helps new managers visualize where their time is going vs. where it *should* go.

```go
package main

import (
	"fmt"
)

type Activity string

const (
	Coding   Activity = "Coding"
	Meeting  Activity = "Meeting"
	Planning Activity = "Planning"
	Coaching Activity = "Coaching"
)

type Schedule struct {
	Role       string
	Activities map[Activity]int // Percentage
}

func Audit(s Schedule) {
	fmt.Printf("--- Time Audit for %s ---\n", s.Role)
	
	if s.Role == "Engineering Manager" {
		if s.Activities[Coding] > 20 {
			fmt.Println("⚠️  WARNING: You are coding too much (>20%). Delegate!")
		}
		if s.Activities[Coaching] < 30 {
			fmt.Println("⚠️  WARNING: You are neglecting your team (<30% Coaching).")
		}
	}
	
	fmt.Println("Analysis Complete.")
}

func main() {
	newEM := Schedule{
		Role: "Engineering Manager",
		Activities: map[Activity]int{
			Coding:   50, // Clinging to old role
			Meeting:  30,
			Coaching: 10,
			Planning: 10,
		},
	}

	Audit(newEM)
}
```

## Interview Questions

### Q: "How do you help a new manager who is overwhelmed?"
**A:**
*   **Normalize the struggle**: "Everyone feels like an imposter for the first 6 months."
*   **Force Delegation**: "Pick one meeting or task you own and give it to a Senior Engineer."
*   **Calendar Audit**: Look at their week. Are they attending meetings they don't need to?

### Q: "What is the hardest part of moving from Senior Dev to Lead?"
**A:**
*   **Feedback**: Giving critical feedback to people who were your peers/friends yesterday.
*   **Solution**: Establish the new relationship explicitly. "I am your manager now, which means my job is to help you grow, sometimes by giving tough news."

### Q: "Can a manager go back to being an IC?"
**A:**
*   **Yes!** It's called the "Pendulum."
*   Many great engineers try management, hate it, and return to IC work with *better* empathy for leadership. It should be celebrated, not seen as failure.
