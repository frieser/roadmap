# Responsibility Shifts

## Summary
Responsibility Shifts happen when a team takes ownership of a new domain (e.g., "You now own the Payment Service") or hands one off. This is distinct from a reorg; the team stays the same, but the *work* changes. Clear "Transfer of Ownership" protocols are essential to prevent "Zombie Services" (services nobody owns).

## Detailed Explanation

### 1. The Handover Ritual
*   You cannot just change the name in a spreadsheet.
*   **Checklist**:
    *   Code Walkthrough.
    *   Runbook / Documentation review.
    *   Alert routing update (PagerDuty).
    *   Backlog grooming (delete stale tickets).

### 2. Cognitive Load Management
*   If you give a team a new responsibility, you must take one away (or hire more people).
*   *Anti-Pattern*: "Just add it to your plate." This leads to burnout and quality drops.

### 3. Service Tiering
*   Not all services are equal.
*   **Tier 1**: Critical path (Checkout). Needs 24/7 on-call.
*   **Tier 2**: Internal tools. N/A on weekends.
*   *Shift*: Moving from owning Tier 2 to Tier 1 requires a culture shift (On-call discipline).

## Go Code Example: Ownership Registry
This tool tracks service ownership and verifies that contact details are valid.

```go
package main

import (
	"fmt"
)

type Service struct {
	Name      string
	OwnerTeam string
	OnCall    string // PagerDuty ID
}

type OwnershipRegistry struct {
	Services []Service
}

func (r OwnershipRegistry) Audit() {
	orphans := 0
	fmt.Println("--- Ownership Audit ---")
	for _, s := range r.Services {
		if s.OwnerTeam == "Unassigned" {
			fmt.Printf("👻 ZOMBIE SERVICE: %s has no owner!\n", s.Name)
			orphans++
		} else {
			fmt.Printf("✅ %s owned by %s\n", s.Name, s.OwnerTeam)
		}
	}
	
	if orphans > 0 {
		fmt.Println("🚨 ACTION REQUIRED: Assign owners or decommission.")
	}
}

func main() {
	registry := OwnershipRegistry{
		Services: []Service{
			{"AuthService", "IdentityTeam", "PD-123"},
			{"LegacyBilling", "Unassigned", ""}, // Team left
			{"FrontendApp", "WebTeam", "PD-456"},
		},
	}

	registry.Audit()
}
```

## Interview Questions

### Q: "A team refuses to take ownership of a legacy service. What do you do?"
**A:**
*   **Understand Why**: Is it written in a language they don't know? Is it a mess?
*   **Sweeten the Deal**: "If you take this, you get to rewrite it in Go next quarter."
*   **Resource It**: "I will give you headcount for 1 contractor to maintain this while you focus on new stuff."

### Q: "How do you ensure a smooth handover?"
**A:**
*   **Shadowing**: The receiving team shadows the owning team for 2 weeks.
*   **Reverse Shadowing**: The receiving team drives, the old team watches.
*   **Cutover**: Formal sign-off. "You have the con."

### Q: "What is the 'You Build It, You Run It' philosophy?"
**A:**
*   DevOps principle. If devs know they will be woken up at 3am for bugs, they write better code.
*   Shifting responsibility from Ops to Devs improves quality.
