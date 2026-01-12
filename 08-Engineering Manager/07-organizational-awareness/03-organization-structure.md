# Organization Structure

## Summary
Organization Structure defines the lines of authority, communication, and information flow. Understanding the official org chart—and the *unofficial* influence network—is key to getting things done. For EMs, this means understanding Matrix organizations, centralized vs. decentralized models, and how to scale teams.

## Detailed Explanation

### 1. Functional vs. Divisional vs. Matrix
*   **Functional**: Grouped by skill (Engineering Dept, Sales Dept).
    *   *Pro*: Deep expertise, consistent standards.
    *   *Con*: Silos, slow handoffs ("Throw it over the wall").
*   **Divisional**: Grouped by product/market (Uber Eats Div, Uber Rides Div).
    *   *Pro*: Speed, customer focus.
    *   *Con*: Duplication of effort (two teams building auth).
*   **Matrix**: Reporting to two bosses (e.g., Functional Manager for career, Product Lead for daily work).
    *   *Pro*: Balance.
    *   *Con*: Confusion, conflict ("Who do I listen to?").

### 2. Centralized vs. Decentralized
*   **Centralized**: Decisions made at the top. High consistency, low autonomy.
*   **Decentralized**: Decisions made at the edge. High speed, potential chaos.
*   *Trend*: Modern tech uses **"Aligned Autonomy"** (Spotify Model). Centralize *strategy/standards*, decentralize *execution*.

### 3. Span of Control
*   The number of direct reports a manager has.
*   **Ideal for EM**: 6-8.
*   **Too Low (<4)**: Micromanagement risk.
*   **Too High (>10)**: Neglect risk. Manager becomes a bottleneck.

## Go Code Example: Org Chart Traversal
This example models an Org Chart as a tree structure and finds the "Common Manager" (Lowest Common Ancestor) to resolve conflicts between two employees.

```go
package main

import (
	"fmt"
)

type Employee struct {
	Name    string
	Manager *Employee
}

// FindChain returns the reporting chain up to the CEO
func (e *Employee) FindChain() []*Employee {
	chain := []*Employee{e}
	curr := e
	for curr.Manager != nil {
		curr = curr.Manager
		chain = append(chain, curr)
	}
	return chain
}

// FindEscalationPoint finds the first common manager
func FindEscalationPoint(e1, e2 *Employee) *Employee {
	chain1 := e1.FindChain()
	chain2 := e2.FindChain()

	// Compare chains to find first intersection
	for _, m1 := range chain1 {
		for _, m2 := range chain2 {
			if m1 == m2 {
				return m1
			}
		}
	}
	return nil
}

func main() {
	ceo := &Employee{Name: "CEO", Manager: nil}
	cto := &Employee{Name: "CTO", Manager: ceo}
	vpEng := &Employee{Name: "VP Eng", Manager: cto}
	
	managerA := &Employee{Name: "Manager A", Manager: vpEng}
	managerB := &Employee{Name: "Manager B", Manager: vpEng}
	
	dev1 := &Employee{Name: "Alice", Manager: managerA}
	dev2 := &Employee{Name: "Bob", Manager: managerB}

	// Conflict between Alice and Bob needs to be resolved by...
	resolver := FindEscalationPoint(dev1, dev2)
	
	fmt.Printf("Conflict between %s and %s\n", dev1.Name, dev2.Name)
	if resolver != nil {
		fmt.Printf("Escalation Point: %s\n", resolver.Name)
	}
}
```

## Interview Questions

### Q: "What are the pros and cons of a Matrix organization?"
**A:**
*   **Pros**: Efficient resource usage (experts move to where needed), maintains functional excellence while delivering projects.
*   **Cons**: "Two Bosses" problem. Power struggles between Functional and Project managers.
*   **Mitigation**: Clear R&R (Roles and Responsibilities). Functional Manager owns "How" (Quality, Career). Product Lead owns "What" (Priorities).

### Q: "When should you introduce a 'layer' of management?"
**A:**
*   When the Span of Control exceeds 8-10.
*   Signs: You are skipping 1:1s, you don't know what your team is working on, hiring has stalled because you are too busy.
*   *Action*: Promote a Tech Lead to EM or split the team.

### Q: "How do you break down silos between Engineering and Sales?"
**A:**
*   **Embed**: Have engineers sit in on sales calls (empathy).
*   **Shared Goals**: Align incentives. If Sales only cares about revenue and Eng only cares about tech debt, they will fight. Create a shared OKR around "Customer Satisfaction" or "Time to Value."
