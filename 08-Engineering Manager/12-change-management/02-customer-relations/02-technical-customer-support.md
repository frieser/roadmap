# Technical Customer Support

## Summary
Technical Customer Support is the escalation path for issues that Tier 1 Support cannot solve. For EMs, managing this relationship is vital. If engineers are pulled into too many support tickets, roadmap velocity dies. If they are pulled into too few, quality suffers.

## Detailed Explanation

### 1. The Support Funnel
*   **Tier 1 (Frontline)**: "Have you tried turning it off and on?" (Scripts).
*   **Tier 2 (Technical Support)**: Reading logs, reproducing bugs, SQL queries.
*   **Tier 3 (Engineering)**: Changing code. This is the most expensive resource.

### 2. Rotations vs. Interruptions
*   **The Problem**: Random interruptions kill "Flow State."
*   **The Solution**: **On-Call Rotation**. One engineer is the "Batman/Hero" for the week. They handle all interruptions. Everyone else focuses on roadmap work.

### 3. Runbooks
*   Documentation is the only way to scale support.
*   Every time an engineer solves a Tier 3 issue, they must write a Runbook entry so Tier 2 can solve it next time. "Shift Left" the knowledge.

## Go Code Example: Ticket Escalation Logic
This example models an automated escalation policy based on SLA (Service Level Agreement) deadlines.

```go
package main

import (
	"fmt"
	"time"
)

type Ticket struct {
	ID        string
	Severity  string // P1, P2, P3
	Created   time.Time
	Status    string
}

func CheckEscalation(t Ticket) string {
	now := time.Now()
	age := now.Sub(t.Created)

	// SLA Rules
	// P1: 1 hour
	// P2: 24 hours
	// P3: 72 hours
	
	if t.Severity == "P1" {
		if age > 1*time.Hour {
			return "🚨 BREACH: Page Engineering Manager!"
		} else if age > 30*time.Minute {
			return "⚠️ WARN: Notify On-Call Engineer"
		}
	} else if t.Severity == "P2" {
		if age > 24*time.Hour {
			return "🚨 BREACH: Notify Product Owner"
		}
	}

	return "✅ Within SLA"
}

func main() {
	// Simulate a ticket created 45 mins ago
	p1Ticket := Ticket{
		ID: "T-101", Severity: "P1", Created: time.Now().Add(-45 * time.Minute),
	}

	// Simulate a ticket created 2 days ago
	p2Ticket := Ticket{
		ID: "T-102", Severity: "P2", Created: time.Now().Add(-48 * time.Hour),
	}

	fmt.Printf("Ticket %s: %s\n", p1Ticket.ID, CheckEscalation(p1Ticket))
	fmt.Printf("Ticket %s: %s\n", p2Ticket.ID, CheckEscalation(p2Ticket))
}
```

## Interview Questions

### Q: "Engineers are complaining about doing support work. What do you do?"
**A:**
*   **Validate**: "I know it sucks to stop coding."
*   **Shield**: Implement the "Batman" rotation so only 1 person suffers at a time.
*   **Root Cause**: "Why are we getting so many tickets?" Fix the bugs/UX so the tickets stop coming. This turns support pain into motivation for quality.

### Q: "How do you measure the quality of technical support?"
**A:**
*   **CSAT (Customer Satisfaction)**: "How was my help?"
*   **Time to Resolution**: Speed.
*   **Reopen Rate**: Did we actually fix it, or did they come back?

### Q: "What is an SLA and why does Engineering care?"
**A:**
*   **Service Level Agreement**: A contractual promise (e.g., "99.9% uptime").
*   **Impact**: If we breach it, we pay money back. It gives Engineering the budget to prioritize stability over new features.
