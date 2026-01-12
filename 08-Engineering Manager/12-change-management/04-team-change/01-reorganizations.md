# Reorganizations

## Summary
Reorganizations ("Reorgs") are structural changes to teams, reporting lines, or departments. While aimed at improving efficiency or alignment, they are stressful and disruptive. Successful reorgs are executed quickly, communicated clearly, and focus on the *future state* benefits while acknowledging the *present state* pain.

## Detailed Explanation

### 1. Why Reorg?
*   **Strategy Shift**: Company pivots from "Growth" to "Profitability" (merging sales/marketing).
*   **Scale**: Teams grew too big (>10 people) and need splitting (Mitosis).
*   **Efficiency**: Removing silos or middle management layers.

### 2. The Process
1.  **Design**: Leadership defines the new structure in secret (to avoid rumor mill).
2.  **Selection**: Slotting people into boxes.
3.  **Announcement**: The "Band-Aid" moment. Tell everyone at once.
4.  **Stabilization**: The 3-6 months after, where the actual work happens.

### 3. Conway's Law Impact
*   Changing the team structure *will* change the software architecture.
*   If you split the Backend Team into "Core" and "API," your monolith will eventually split into two services.

## Go Code Example: Team Splitter (Mitosis)
This example simulates splitting a large team into two smaller squads based on load balancing skill sets.

```go
package main

import (
	"fmt"
)

type Engineer struct {
	Name  string
	Role  string // Frontend, Backend, SRE
	Level int    // 1-5
}

type Squad struct {
	Name    string
	Members []Engineer
}

func (s Squad) PowerLevel() int {
	total := 0
	for _, m := range s.Members {
		total += m.Level
	}
	return total
}

func SplitTeam(bigTeam []Engineer) (Squad, Squad) {
	squadA := Squad{Name: "Alpha"}
	squadB := Squad{Name: "Bravo"}

	// Simple Round Robin Distribution
	for i, e := range bigTeam {
		if i%2 == 0 {
			squadA.Members = append(squadA.Members, e)
		} else {
			squadB.Members = append(squadB.Members, e)
		}
	}
	return squadA, squadB
}

func main() {
	bigTeam := []Engineer{
		{"Alice", "Backend", 5}, {"Bob", "Frontend", 4},
		{"Charlie", "Backend", 3}, {"Dave", "SRE", 4},
		{"Eve", "Frontend", 2}, {"Frank", "Backend", 3},
	}

	a, b := SplitTeam(bigTeam)

	fmt.Printf("Squad A Power: %d | Members: %d\n", a.PowerLevel(), len(a.Members))
	fmt.Printf("Squad B Power: %d | Members: %d\n", b.PowerLevel(), len(b.Members))
}
```

## Interview Questions

### Q: "How do you handle an employee who is angry about their new manager after a reorg?"
**A:**
*   **Listen**: Let them vent. Validation lowers temperature.
*   **Explain the Why**: "We moved you because your skill set is critical for the new Mobile initiative."
*   **Commit**: "Give it 3 months. If it's still terrible, we can look at internal transfers."

### Q: "Why do reorgs often fail?"
**A:**
*   **Lack of Clarity**: People don't know their new roles.
*   **Frozen Middle**: Leadership decides, but middle managers (EMs) aren't bought in, so they sabotage it passively.
*   **Constant Shuffle**: Reorging every 6 months prevents teams from ever reaching "Performing" stage (Tuckman model).

### Q: "What is the 'Bus Factor' risk during a reorg?"
**A:**
*   If the only person who knows the Legacy Billing Code gets moved to the AI Team, you have created a ticking time bomb.
*   *Mitigation*: Documentation and handover periods are mandatory.
