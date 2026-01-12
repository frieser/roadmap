# Culture Building

## Summary
Culture Building is the intentional design of the "operating system" of a team. It is defined not by what you write on the wall, but by **what you reward** and **what you tolerate**. A strong culture reduces decision-making overhead, attracts talent, and creates psychological safety.

## Detailed Explanation

### 1. Values vs. Virtues
*   **Values**: Abstract concepts (e.g., "Integrity," "Innovation").
*   **Virtues**: Actionable behaviors (e.g., "We admit mistakes openly," "We disagree and commit").
*   *Task*: Translate values into specific behaviors you can hire and fire for.

### 2. Psychological Safety (Project Aristotle)
Google found this to be the #1 predictor of team success. It is the belief that you won't be punished or humiliated for speaking up with ideas, questions, concerns, or mistakes.
*   **How to build it**: Admit your own fallibility ("I messed up"), encourage questions, and respond to bad news with curiosity, not anger.

### 3. Rituals and Artifacts
Culture is reinforced through rituals:
*   **Demos**: Celebrate shipping.
*   **Post-Mortems**: Celebrate learning from failure.
*   **Onboarding**: The first week sets the cultural baseline for new hires.

## Go Code Example: Cultural Values Game
This example is a simple simulation of a "Kudos" system where team members reward each other for exhibiting specific cultural values.

```go
package main

import (
	"fmt"
	"sort"
)

type Value string

const (
	Ownership   Value = "Ownership"
	Curiosity   Value = "Curiosity"
	Teamwork    Value = "Teamwork"
	Transparency Value = "Transparency"
)

type Kudo struct {
	From    string
	To      string
	Value   Value
	Message string
}

type CultureBot struct {
	Kudos []Kudo
}

func (b *CultureBot) GiveKudo(from, to string, val Value, msg string) {
	k := Kudo{From: from, To: to, Value: val, Message: msg}
	b.Kudos = append(b.Kudos, k)
	fmt.Printf("🏆 %s gave %s a kudo for %s: \"%s\"\n", from, to, val, msg)
}

func (b *CultureBot) ShowLeaderboard() {
	counts := make(map[string]int)
	for _, k := range b.Kudos {
		counts[k.To]++
	}

	// Sort (simplified)
	var names []string
	for n := range counts {
		names = append(names, n)
	}
	sort.Slice(names, func(i, j int) bool {
		return counts[names[i]] > counts[names[j]]
	})

	fmt.Println("\n--- Culture Champions ---")
	for _, n := range names {
		fmt.Printf("%s: %d Kudos\n", n, counts[n])
	}
}

func main() {
	bot := CultureBot{}

	bot.GiveKudo("Alice", "Bob", Teamwork, "Helped me debug the prod issue late at night.")
	bot.GiveKudo("Charlie", "Bob", Ownership, "Took full responsibility for the migration.")
	bot.GiveKudo("Bob", "Alice", Transparency, "Shared the incident report honestly.")
	bot.GiveKudo("Dave", "Alice", Curiosity, "Asked great questions during the design review.")

	bot.ShowLeaderboard()
}
```

## Interview Questions

### Q: "How do you define your team's culture?"
**A:**
*   **Focus**: Be specific. "We are a culture of high autonomy and high alignment. We value shipping fast over perfect code, and we treat production outages as learning opportunities, not blame games."

### Q: "How do you change a toxic team culture?"
**A:**
*   **Diagnose**: Is it a systems problem (incentives) or a people problem (bad apple)?
*   **Reset Expectations**: Explicitly state the new norms.
*   **Model Behavior**: You must live the new culture first.
*   **Remove Detractors**: If someone refuses to adapt (even a high performer), they must leave.

### Q: "What is 'Psychological Safety' and how do you foster it?"
**A:**
*   **Definition**: The ability to take risks without fear of negative consequences to self-image/career.
*   **Action**: I foster it by asking "What did we learn?" instead of "Who broke it?", by admitting my own mistakes first, and by ensuring everyone speaks in meetings.
