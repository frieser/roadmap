# Budget Planning

## Summary
Budget Planning is the translation of engineering strategy into financial terms. For an Engineering Manager, it involves estimating the costs of headcount (people), infrastructure (cloud), and tools (SaaS) required to deliver the roadmap. A well-planned budget ensures the team has the resources to succeed without overspending.

## Detailed Explanation

### 1. Components of an Engineering Budget
*   **Headcount (OpEx)**: Salaries, benefits, taxes, and recruiting fees. This is usually ~70-80% of the budget.
*   **Infrastructure (COGS or OpEx)**: Cloud costs (AWS, GCP), data centers. If it serves customers directly, it's often Cost of Goods Sold (COGS).
*   **Software & Tools (OpEx)**: Licenses for IDEs, Jira, Slack, CI/CD, Monitoring (Datadog/New Relic).
*   **Services**: Contractors, consultants, training budgets.

### 2. CapEx vs. OpEx
*   **CapEx (Capital Expenditure)**: Money spent on physical assets (servers, laptops) or *sometimes* R&D for new products (capitalized software). Depreciated over time.
*   **OpEx (Operating Expenditure)**: Day-to-day running costs (Cloud bills, SaaS subscriptions, Salaries).
*   *Why it matters*: Finance teams often prefer CapEx for tax reasons, but modern DevOps moves everything to OpEx (Rent > Buy).

### 3. The Planning Process
1.  **Bottom-Up**: "What do we need to build X?" (Summing up costs).
2.  **Top-Down**: "Here is your envelope ($Y million). Make it work."
3.  **Buffer**: Always add 10-20% for unknown unknowns.

## Go Code Example: Budget Forecaster
This example models a simple budget planner that calculates monthly burn rates and projects annual spend.

```go
package main

import (
	"fmt"
)

type Category string

const (
	Headcount      Category = "Headcount"
	Infrastructure Category = "Infrastructure"
	Software       Category = "Software"
)

type LineItem struct {
	Name     string
	Category Category
	Monthly  float64
}

type Budget struct {
	Items []LineItem
}

func (b Budget) CalculateAnnual() map[Category]float64 {
	totals := make(map[Category]float64)
	for _, item := range b.Items {
		totals[item.Category] += item.Monthly * 12
	}
	return totals
}

func main() {
	budget := Budget{
		Items: []LineItem{
			{"Senior Engineer", Headcount, 15000},
			{"Junior Engineer", Headcount, 8000},
			{"AWS Bill", Infrastructure, 2500},
			{"Datadog", Software, 500},
			{"GitHub Enterprise", Software, 200},
		},
	}

	totals := budget.CalculateAnnual()
	grandTotal := 0.0

	fmt.Println("--- Annual Engineering Budget ---")
	for cat, amount := range totals {
		fmt.Printf("%s: $%.2f\n", cat, amount)
		grandTotal += amount
	}
	
	fmt.Println("---------------------------------")
	fmt.Printf("TOTAL: $%.2f\n", grandTotal)
}
```

## Interview Questions

### Q: "How do you handle a request to cut the budget by 10%?"
**A:**
*   **Prioritize**: Identify non-essential spend (e.g., unused SaaS seats, dev environments running on weekends).
*   **Trade-offs**: Communicate impact clearly. "If we cut headcount, project Z will be delayed by 3 months."
*   **Efficiency**: Look for cloud cost optimizations (Reserved Instances, Spot Instances) before cutting people.

### Q: "Explain the difference between CapEx and OpEx to a junior engineer."
**A:**
*   **CapEx**: Buying a house. You pay upfront, own it, and its value goes down over time (depreciation). Example: Buying a rack of servers.
*   **OpEx**: Renting an apartment. You pay monthly, don't own it, but have flexibility to leave. Example: Paying AWS EC2 hourly rates.

### Q: "Why is headcount usually the biggest line item?"
**A:**
*   Software engineering is knowledge work. The value is created by humans, not machines. While cloud bills can be high, the cost of a full team of senior engineers (salary + equity + overhead) dwarfs infrastructure costs for most SaaS companies.
