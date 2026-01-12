# Defining and Enforcing Values

## Summary
Team Values are the "operating principles" that guide decision-making when the manager isn't in the room. Defining them is easy; **enforcing** them is hard. Values are worthless if they are not actionable or if violations are tolerated. Effective values are not generic words like "Integrity" but specific behaviors like "Disagree and Commit."

## Detailed Explanation

### 1. Values vs. Virtues (Ben Horowitz)
*   **Values**: Abstract ideals (e.g., "Excellence").
*   **Virtues**: What you *do* (e.g., "We write tests for every bug fix").
*   *Action*: Translate every value into a "We do X" and "We do not do Y" statement.

### 2. The "Weaponization" of Values
*   Values can be used to silence dissent (e.g., "Be Nice" can suppress necessary conflict).
*   *Solution*: Balance values. "Radical Candor" balances "Empathy" with "Direct Feedback."

### 3. Enforcing Values
*   **Hiring**: Interview for values (e.g., "Tell me about a time you disagreed with a decision").
*   **Firing**: The hardest part. You must fire high performers who violate values ("Brilliant Jerks"). If you don't, the value is a lie.
*   **Rewarding**: Promote people who exemplify the values.

## Go Code Example: Values Linter
This conceptual example checks if a team's decisions align with their stated values.

```go
package main

import (
	"fmt"
)

type Value string

const (
	Transparency Value = "Default to Open"
	Velocity     Value = "Move Fast"
	Quality      Value = "Don't Break Prod"
)

type Decision struct {
	Description string
	AlignedWith []Value
	Violates    []Value
}

func ReviewDecision(d Decision) {
	fmt.Printf("Decision: %s\n", d.Description)
	
	if len(d.Violates) > 0 {
		fmt.Println("❌ BLOCKED: Violates core values:")
		for _, v := range d.Violates {
			fmt.Printf("   - %s\n", v)
		}
		return
	}

	fmt.Println("✅ APPROVED: Aligns with values:")
	for _, v := range d.AlignedWith {
		fmt.Printf("   + %s\n", v)
	}
}

func main() {
	// Scenario: Launching a feature without tests to hit a deadline
	rushJob := Decision{
		Description: "Ship feature X by Friday (skipping QA)",
		AlignedWith: []Value{Velocity},
		Violates:    []Value{Quality},
	}

	// Scenario: Publishing a post-mortem publicly
	publicIncident := Decision{
		Description: "Publish RCA blog post",
		AlignedWith: []Value{Transparency},
		Violates:    []Value{},
	}

	ReviewDecision(rushJob)
	fmt.Println("---")
	ReviewDecision(publicIncident)
}
```

## Interview Questions

### Q: "How do you handle a 'Brilliant Jerk'?"
**A:**
*   **Diagnose**: Is it a lack of awareness or a choice?
*   **Feedback**: Give specific examples of the behavior and its impact on the team.
*   **Ultimatum**: "Your technical output is great, but your behavior is failing the 'Teamwork' value. This is a job requirement."
*   **Exit**: If they don't change, fire them. The team's performance usually goes *up* after they leave.

### Q: "What is a value you have introduced to a team?"
**A:**
*   **Example**: "Strong Opinions, Loosely Held."
*   **Why**: The team was paralyzed by consensus-seeking. This value encouraged debate but also allowed us to move forward when data was ambiguous.

### Q: "How do you keep values from becoming just posters on the wall?"
**A:**
*   **Rituals**: We reference them in code reviews ("This violates 'Simple over Complex'") and in performance reviews.
*   **Storytelling**: "Remember when Alice stayed late to help Bob? That was 'Teamwork' in action."
