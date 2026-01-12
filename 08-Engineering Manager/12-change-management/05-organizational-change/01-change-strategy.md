# Change Strategy

## Summary
Change Strategy is the high-level plan for how an organization moves from State A to State B. It is not just about the "What" (technical migration) but the "How" (people adaptation). A good strategy selects the right *pace* (Revolution vs. Evolution) and the right *levers* (Structure, Process, or Technology) to ensure the change sticks.

## Detailed Explanation

### 1. Types of Change
*   **Developmental**: Improving existing skills/processes (e.g., "We are adopting Go 1.20"). Low pain.
*   **Transitional**: Replacing something known with something new (e.g., "We are moving to Jira"). Medium pain.
*   **Transformational**: Culture/Identity shift (e.g., "We are becoming an AI-first company"). High pain, high risk.

### 2. Levers of Change (The Golden Triangle)
*   **People**: Training, hiring, firing.
*   **Process**: Rules, workflows, policies.
*   **Technology**: Tools, automation.
*   *Rule*: Changing Tech without changing People/Process creates "Shelfware" (tools nobody uses).

### 3. Kotter's 8 Steps
A classic framework:
1.  Create Urgency.
2.  Form a Coalition.
3.  Create a Vision.
4.  Communicate.
5.  Empower Action (Remove barriers).
6.  Quick Wins.
7.  Build on the Change.
8.  Anchor in Culture.

## Go Code Example: Strategy Selector
This logic helps select the right strategy based on urgency and risk.

```go
package main

import (
	"fmt"
)

type Context struct {
	Urgency      int // 1-10
	CultureHealth int // 1-10 (Trust level)
	Complexity   int // 1-10
}

func SelectStrategy(c Context) string {
	if c.Urgency > 8 {
		if c.CultureHealth > 7 {
			return "CRISIS MODE: Top-down mandate (Team trusts you)"
		}
		return "DICTATORSHIP: Top-down mandate (expect attrition)"
	}

	if c.Complexity > 7 {
		return "PILOT PROGRAM: Isolate risk, learn, then scale."
	}

	if c.CultureHealth > 5 {
		return "GRASSROOTS: Empower teams to change themselves."
	}

	return "MANAGED ROLLOUT: Standard training and scheduled adoption."
}

func main() {
	// Scenario: Security Breach fix
	crisis := Context{Urgency: 10, CultureHealth: 5, Complexity: 3}
	fmt.Println("Scenario 1:", SelectStrategy(crisis))

	// Scenario: Moving to Microservices
	migration := Context{Urgency: 3, CultureHealth: 8, Complexity: 9}
	fmt.Println("Scenario 2:", SelectStrategy(migration))
}
```

## Interview Questions

### Q: "When do you use a 'Big Bang' strategy vs. 'Incremental'?"
**A:**
*   **Incremental**: Almost always. It lowers risk and allows feedback loops.
*   **Big Bang**: Only when the systems cannot coexist (e.g., changing the legal entity, or a breaking API change that cannot be versioned).

### Q: "What is the biggest mistake EMs make in change strategy?"
**A:**
*   **Under-communicating**: Assuming "I said it once in All-Hands, so everyone knows."
*   **Ignoring the 'Dip'**: Expecting productivity to stay linear during the change. It *will* drop. Plan for it.

### Q: "How do you align strategy with culture?"
**A:**
*   If the culture is "Consensus-driven," a "Top-down" strategy will fail. You must build the coalition first.
*   If the culture is "Move Fast," a "Bureaucratic" strategy will be ignored.
