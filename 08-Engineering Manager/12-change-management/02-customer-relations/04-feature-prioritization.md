# Feature Prioritization

## Summary
Feature Prioritization is the process of deciding what to build next when you have infinite ideas and finite resources. It is the core conflict resolution mechanism between Engineering (Debt/Refactoring), Product (New Features), and Sales (Customer Requests).

## Detailed Explanation

### 1. Frameworks
*   **RICE Score**: Reach, Impact, Confidence, Effort. (Data-driven).
*   **MoSCoW**: Must have, Should have, Could have, Won't have. (Simple).
*   **Kano Model**: Basic Needs (must be there), Performance (more is better), Delighters (surprise).

### 2. The "Portfolio" Approach (Capacity Allocation)
Don't prioritize everything in one bucket. Allocate capacity percentages:
*   **50% Innovation**: New features (Product-led).
*   **30% KTLO (Keep the Lights On)**: Maintenance, bug fixes, updates.
*   **20% Strategic Debt**: Refactoring, architectural improvements (Eng-led).
*   *Why*: Ensures long-term health while shipping value.

### 3. Cost of Delay (CD3)
*   "If we delay this feature by 1 month, how much money do we lose?"
*   Prioritize the item with the highest *Cost of Delay divided by Duration*.

## Go Code Example: RICE Calculator
This tool helps rank backlog items objectively.

```go
package main

import (
	"fmt"
	"sort"
)

type Feature struct {
	Name       string
	Reach      float64 // Users affected
	Impact     float64 // 3=Massive, 2=High, 1=Medium, 0.5=Low
	Confidence float64 // % (0.0 - 1.0)
	Effort     float64 // Person-months
}

func (f Feature) RICE() float64 {
	return (f.Reach * f.Impact * f.Confidence) / f.Effort
}

func main() {
	backlog := []Feature{
		{"Dark Mode", 1000, 1.0, 1.0, 2.0},      // Low impact, low effort
		{"SSO", 50, 3.0, 1.0, 4.0},              // High impact, high effort, low reach
		{"Checkout Optimization", 1000, 3.0, 0.8, 3.0}, // High everything
	}

	// Sort by RICE Score descending
	sort.Slice(backlog, func(i, j int) bool {
		return backlog[i].RICE() > backlog[j].RICE()
	})

	fmt.Println("--- Prioritized Backlog (RICE) ---")
	for i, f := range backlog {
		fmt.Printf("%d. %s (Score: %.1f)\n", i+1, f.Name, f.RICE())
	}
}
```

## Interview Questions

### Q: "Product wants 100% of the sprint for new features. Engineering wants 50% for refactoring. How do you resolve this?"
**A:**
*   **Visualize**: Show the "Debt Wall." If we don't refactor, velocity will drop to zero next quarter.
*   **Compromise**: Agree on a fixed allocation (e.g., 20%).
*   **Sell the Value**: "Refactoring X means Feature Y will take 3 days instead of 10 days next month."

### Q: "How do you say 'No' to the CEO?"
**A:**
*   **"Yes, and..."**: "Yes, we can do that, *and* it means we have to drop Project Z."
*   **Trade-offs**: Never say "No" outright. Present options. "We have 3 devs. We can do A or B. Which is more important for the Q1 goal?"

### Q: "What is 'Weighted Shortest Job First' (WSJF)?"
**A:**
*   A SAFe (Scaled Agile) metric similar to Cost of Delay.
*   It prioritizes high-value, short-duration jobs to maximize flow.
