# Resistance Management

## Summary
Resistance Management is dealing with the pushback against change. Resistance is not always bad; it often highlights flaws in the plan. However, unexplained or emotional resistance can derail a project. EMs must distinguish between "I don't understand," "I can't do it," and "I won't do it."

## Detailed Explanation

### 1. The Resistance Pyramid (Galpin)
1.  **"I don't know"** (Lack of Awareness). *Fix: Communication.*
2.  **"I can't"** (Lack of Ability). *Fix: Training/Resources.*
3.  **"I won't"** (Lack of Willingness). *Fix: Incentives/Coercion.*

### 2. Active vs. Passive Resistance
*   **Active**: Arguing in meetings. (Good! You can address it).
*   **Passive**: Nodding in meetings, then doing nothing. (Dangerous. Hard to detect).

### 3. Converting Detractors
*   **Listen**: Usually, they just want to be heard. "I'm worried about X."
*   **Involve**: "Great point. Can you lead the task force to solve X?"
*   **Contain**: If they are toxic, isolate them from the rest of the team so resistance doesn't spread.

## Go Code Example: Resistance Classifier
This example categorizes user feedback into resistance types to suggest a mitigation strategy.

```go
package main

import (
	"fmt"
)

type Feedback struct {
	User    string
	Comment string
	Type    string // "Confusion", "Ability", "Willingness"
}

func Mitigate(f Feedback) string {
	switch f.Type {
	case "Confusion":
		return "Explain the 'Why' (Town Hall / FAQ)"
	case "Ability":
		return "Provide Training / Workshops"
	case "Willingness":
		return "1:1 Coaching / Incentives / Performance Mgmt"
	default:
		return "Listen"
	}
}

func main() {
	feedbacks := []Feedback{
		{"Alice", "I don't see why we need to change.", "Confusion"},
		{"Bob", "I don't know how to use Kubernetes.", "Ability"},
		{"Charlie", "I refuse to use this tool, it sucks.", "Willingness"},
	}

	fmt.Println("--- Resistance Management Plan ---")
	for _, f := range feedbacks {
		fmt.Printf("User: %s | Issue: %s -> Action: %s\n", f.User, f.Type, Mitigate(f))
	}
}
```

## Interview Questions

### Q: "How do you handle a Senior Engineer who refuses to adopt the new coding standard?"
**A:**
*   **1:1**: "Help me understand your concern."
*   **Valid Objection?**: If they have a valid technical point, adapt the standard.
*   **Invalid Objection?**: "I hear you, but the team decided X. As a Senior, I need you to commit. Continued refusal undermines the team."
*   **Escalation**: It becomes a performance issue (Insubordination/Values violation).

### Q: "What is the 'Innovation Adoption Curve'?"
**A:**
*   **Innovators/Early Adopters**: Will try anything.
*   **Early Majority**: Need proof.
*   **Late Majority**: Need peer pressure.
*   **Laggards**: Will never change.
*   *Strategy*: Focus on the Early Majority. Ignore the Laggards until the end.

### Q: "Why is 'Passive Resistance' so dangerous?"
**A:**
*   It creates a false sense of security ("Everyone agreed!").
*   Then the deadline hits and nothing is done.
*   *Fix*: Ask specific questions ("What steps have you taken?") to verify action.
