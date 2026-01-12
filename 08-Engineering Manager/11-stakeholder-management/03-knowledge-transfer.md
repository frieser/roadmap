## Summary
Knowledge Transfer is the deliberate movement of critical information from one person (usually departing or transitioning) to another. It mitigates the risk of "Brain Drain" when key employees leave.

## Detailed Explanation
Effective transfer is structured, not just "let's have coffee."

### The KT Plan
1.  **Inventory**: List everything the expert owns (Code, passwords, relationships).
2.  **Shadowing**: The learner watches the expert.
3.  **Reverse Shadowing**: The expert watches the learner do the task.
4.  **Documentation**: The expert writes/updates docs as they teach.

## Go Code Example
Modeling a `HandoverChecklist` that tracks the transfer of ownership for different system domains.

```go
package main

import "fmt"

type Domain struct {
	Name          string
	CurrentOwner  string
	NewOwner      string
	Status        string // "Not Started", "Shadowing", "Completed"
}

func (d *Domain) Transfer(to string) {
	fmt.Printf("Initiating handover of %s from %s to %s...\n", d.Name, d.CurrentOwner, to)
	d.Status = "Shadowing"
	d.NewOwner = to
}

func (d *Domain) CompleteHandover() {
	if d.Status == "Shadowing" {
		d.Status = "Completed"
		d.CurrentOwner = d.NewOwner
		fmt.Printf("✅ Handover complete. %s is now the owner of %s.\n", d.NewOwner, d.Name)
	}
}

func main() {
	authSystem := Domain{Name: "Auth System", CurrentOwner: "SeniorDev", Status: "Not Started"}
	
	authSystem.Transfer("JuniorDev")
	// ... meaningful time passes ...
	authSystem.CompleteHandover()
}
```

## Interview Questions
**Q: A key engineer is leaving in 2 weeks. What is your plan?**
**A:** Day 1: Brain dump session to list all owned systems. Day 2-10: Pair programming and "Reverse Shadowing" where the *new* owner drives the keyboard while the departing engineer advises. Day 14: Exit interview and revoke access.

**Q: How do you handle knowledge transfer for a complex legacy system?**
**A:** I focus on "Day 2 Operations" (how to restart it, how to debug it) rather than a deep code walk. Code can be read later; operational wisdom ("Don't restart X if Y is running") is what gets lost.
