# Stakeholder Management

## Summary
Stakeholder Management is the art of identifying, analyzing, and influencing the people who can affect or are affected by your project. For EMs, stakeholders include Product, Design, Sales, Execs, and the Team itself. Managing them effectively prevents "swoop and poop" (last-minute executive interference) and ensures alignment.

## Detailed Explanation

### 1. Identification (Who?)
*   **Up**: Executives, Investors. (Care about ROI).
*   **Down**: The Team. (Care about Experience/Workload).
*   **Sideways**: Peers, PMs, Designers. (Care about Collaboration).
*   **Out**: Customers, Partners. (Care about Value).

### 2. The Power/Interest Grid (Mendelow)
*   **High Power / High Interest**: **Manage Closely**. (Your Boss, Key Client). Engage daily/weekly.
*   **High Power / Low Interest**: **Keep Satisfied**. (CFO, Legal). Don't bore them, but ensure compliance.
*   **Low Power / High Interest**: **Keep Informed**. (Support Team, Junior Devs). They can't stop you, but they can be noisy advocates or detractors.
*   **Low Power / Low Interest**: **Monitor**.

### 3. Communication Strategy
*   Different stakeholders need different data.
*   **Execs**: "Green/Red status + Top Risks."
*   **Team**: "Detailed specs + Why we are doing this."

## Go Code Example: Stakeholder Notification Router
This example routes messages to the right channel based on the stakeholder group.

```go
package main

import (
	"fmt"
)

type StakeholderGroup string

const (
	Execs    StakeholderGroup = "Executives"
	Team     StakeholderGroup = "Engineering Team"
	Customers StakeholderGroup = "Customers"
)

func Notify(group StakeholderGroup, event string) {
	fmt.Printf("[%s] ", group)
	
	switch group {
	case Execs:
		fmt.Printf("Email Subject: 'Q3 Update: %s'. Focus: ROI/Timeline.\n", event)
	case Team:
		fmt.Printf("Slack Channel #eng-all: '@here %s'. Focus: Technical details/Action items.\n", event)
	case Customers:
		fmt.Printf("Blog Post/Changelog: 'New Feature: %s'. Focus: Value/Usage.\n", event)
	}
}

func main() {
	event := "Migration to Kubernetes Complete"
	
	Notify(Execs, event)
	Notify(Team, event)
	Notify(Customers, event) // Might frame it as "Improved Reliability"
}
```

## Interview Questions

### Q: "How do you handle a stakeholder who constantly changes requirements?"
**A:**
*   **Visualize the Cost**: "We can change X, but it delays Y by 2 weeks."
*   **Freeze Phase**: "We are in the execution phase. Changes now go to the backlog for v2."
*   **Root Cause**: Why are they changing? Are we showing them demos too late? Move to shorter feedback loops.

### Q: "What do you do if two key stakeholders have conflicting goals?"
**A:**
*   **Don't be the bottleneck**: Put them in a room.
*   **Facilitate**: "Sales wants X, Security wants Y. We can't do both. What is the priority for the business right now?"
*   **Escalate**: If they can't agree, escalate to their shared manager (CEO) with a clear trade-off analysis.

### Q: "How do you manage 'Phantom Stakeholders'?"
**A:**
*   (People who appear only at the end to block).
*   **Map early**: Ask "Who else needs to sign off on this?" repeatedly at the start.
*   **Involve early**: Send them updates even if they don't reply, so they can't say "I didn't know."
