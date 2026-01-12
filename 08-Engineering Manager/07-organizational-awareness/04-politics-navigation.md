# Politics Navigation

## Summary
"Politics" is simply **people making decisions in groups**. It is not inherently evil; it is the mechanism by which resources are allocated and influence is exerted. An EM who ignores politics will fail to get their team the resources, visibility, and protection they need. The goal is to be "politically savvy" (constructive) rather than "political" (manipulative).

## Detailed Explanation

### 1. Types of Power (French & Raven)
*   **Legitimate Power**: "I am the boss." (Weakest form).
*   **Expert Power**: "I know the most about Kubernetes." (Strong in Eng).
*   **Referent Power**: "People like and trust me." (Relationships).
*   **Reward/Coercive Power**: Hiring/Firing/Bonus.

### 2. Social Capital
*   Think of it like a bank account. You make deposits by helping others, delivering on promises, and listening. You make withdrawals when you ask for favors, budget, or exceptions.
*   *Strategy*: Build capital *before* you need it.

### 3. Stakeholder Mapping
Identify who cares about your project and their power/interest.
*   **High Power, High Interest**: Manage Closely (Key Players).
*   **High Power, Low Interest**: Keep Satisfied (Don't annoy them).
*   **Low Power, High Interest**: Keep Informed (Allies).
*   **Low Power, Low Interest**: Monitor.

### 4. The "Meeting before the Meeting"
*   Never walk into a high-stakes decision meeting cold.
*   Socialize your idea ("Nemawashi" in Japanese culture) with key stakeholders individually beforehand to address concerns and secure votes.

## Go Code Example: Stakeholder Analysis Tool
This example categorizes stakeholders into the standard Power/Interest grid and suggests a communication strategy.

```go
package main

import (
	"fmt"
)

type Stakeholder struct {
	Name     string
	Power    int // 1-10
	Interest int // 1-10
}

func (s Stakeholder) DetermineStrategy() string {
	if s.Power >= 6 && s.Interest >= 6 {
		return "Manage Closely (Key Player)"
	} else if s.Power >= 6 && s.Interest < 6 {
		return "Keep Satisfied"
	} else if s.Power < 6 && s.Interest >= 6 {
		return "Keep Informed"
	}
	return "Monitor"
}

func main() {
	stakeholders := []Stakeholder{
		{"CTO", 10, 8},           // Manage Closely
		{"Sales VP", 9, 3},       // Keep Satisfied
		{"Junior Dev", 2, 9},     // Keep Informed
		{"HR Rep", 3, 2},         // Monitor
	}

	fmt.Println("--- Stakeholder Communication Plan ---")
	for _, s := range stakeholders {
		fmt.Printf("%s: %s\n", s.Name, s.DetermineStrategy())
	}
}
```

## Interview Questions

### Q: "How do you handle a situation where another manager is undermining your team?"
**A:**
*   **Assume Positive Intent**: Maybe our goals conflict?
*   **Direct Conversation**: "I noticed X happened. It impacted my team by Y. Can we talk about it?"
*   **Escalation (Last Resort)**: If behavior continues, bring data to the shared manager, focusing on business impact, not personal grievance.

### Q: "Is politics bad?"
**A:**
*   **Bad Politics**: Gossip, backstabbing, hoarding information, prioritizing self over company.
*   **Good Politics**: Building consensus, networking, advocating for your team, aligning disparate goals.
*   *Answer*: "I reject bad politics, but I embrace the necessity of relationship-building to get things done."

### Q: "How do you get buy-in for a project that no one wants to do?"
**A:**
*   **WIIFM (What's In It For Them)**: Frame the project in terms of *their* goals.
    *   To Product: "This refactor will let you ship 2x faster next quarter."
    *   To Sales: "This stability fix prevents the outages that lost you the big deal last week."
