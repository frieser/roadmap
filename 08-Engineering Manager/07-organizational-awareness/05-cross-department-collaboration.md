# Cross-Department Collaboration

## Summary
Engineering does not exist in a vacuum. Cross-Department Collaboration is the practice of working effectively with Product, Design, Sales, Marketing, and Support. Friction often arises from differing incentives (e.g., Sales wants custom features, Eng wants standardization). The EM's role is to align these incentives toward the shared company goal.

## Detailed Explanation

### 1. The Triad (Eng + Product + Design)
*   **Product**: "Why" and "What" (Business Value).
*   **Design**: "How it looks/feels" (User Experience).
*   **Engineering**: "How it works" (Feasibility/Implementation).
*   *Success*: Healthy tension. If one dominates, you get:
    *   *Eng-led*: Over-engineered, poor UX.
    *   *Product-led*: Feature factory, technical debt.
    *   *Design-led*: Beautiful but unbuildable.

### 2. Working with Sales/Marketing
*   **Sales**: Needs dates and certainty.
*   **Eng**: Deals with uncertainty and estimates.
*   *Collaboration*: Involve Eng in sales calls (Sales Engineering) to validate technical feasibility *before* promises are made.
*   *Marketing*: Needs lead time for launches. Eng must communicate delays early.

### 3. Working with Customer Support (CS)
*   CS is the "Canary in the Coal Mine." They see bugs first.
*   *Collaboration*: Rotate engineers into the support queue. Establish a clear escalation path for bugs.

## Go Code Example: Alignment Check
This example simulates checking alignment between departments based on their top priorities.

```go
package main

import (
	"fmt"
)

type Department struct {
	Name     string
	Priority string // e.g., "Speed", "Stability", "Revenue"
}

func CheckAlignment(dept1, dept2 Department) string {
	if dept1.Priority == dept2.Priority {
		return "✅ High Alignment"
	}
	
	// Conflict Matrix
	if (dept1.Priority == "Speed" && dept2.Priority == "Stability") ||
	   (dept1.Priority == "Stability" && dept2.Priority == "Speed") {
		return "⚠️  Classic Tension (Speed vs Stability)"
	}
	
	if dept1.Priority == "Revenue" && dept2.Priority == "Tech Debt" {
		return "🛑 High Conflict (Short-term vs Long-term)"
	}

	return "ℹ️  Divergent Goals (Requires negotiation)"
}

func main() {
	eng := Department{Name: "Engineering", Priority: "Stability"}
	sales := Department{Name: "Sales", Priority: "Speed"}
	product := Department{Name: "Product", Priority: "Stability"} // Aligned with Eng for once!

	fmt.Printf("Eng vs Sales: %s\n", CheckAlignment(eng, sales))
	fmt.Printf("Eng vs Product: %s\n", CheckAlignment(eng, product))
}
```

## Interview Questions

### Q: "Sales sold a feature we don't have. What do you do?"
**A:**
*   **Damage Control**: Talk to the salesperson and the customer. Understand the exact promise.
*   **Assess Feasibility**: Can we hack a "wizard of oz" solution manually? Can we accelerate the roadmap?
*   **Root Cause**: Fix the process. Why did Sales think we had it? Implement a "Technical Review" step in the sales contract process.

### Q: "How do you handle 'Swoop and Poop' management?"
**A:**
*   (When an exec swoops in at the last minute, dumps feedback, and leaves).
*   **Prevention**: Involve stakeholders early and often (Demos).
*   **Documentation**: "We made decision X on date Y because of Z. Changing it now delays launch by W weeks. Do you want to proceed?"

### Q: "How do you build empathy between Engineers and Support?"
**A:**
*   **Support Rotation**: Have every engineer spend 1 day/quarter answering tickets.
*   **Shadowing**: Engineers sit with Support agents to see how they use the internal admin tools (which are usually terrible).
