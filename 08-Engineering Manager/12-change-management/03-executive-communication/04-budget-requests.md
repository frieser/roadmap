# Budget Requests

## Summary
A Budget Request is a specific type of proposal asking for money (Headcount, Tools, Infra). Unlike a strategic proposal, it is heavily financial and competitive (you are competing with Marketing/Sales for the same dollar). The key is to frame the request as an **Investment**, not a **Cost**.

## Detailed Explanation

### 1. Zero-Based Budgeting
*   **Traditional**: "Take last year's budget + 10%."
*   **Zero-Based**: "Start at $0. Justify every dollar."
*   *EM Task*: You must defend even your existing headcount. "Why do we still need 5 devs on the Legacy App?"

### 2. The Business Case for Headcount
*   Don't say: "The team is tired." (Execs think: "Manage them better.")
*   Say: "We have a backlog of $2M in features. With current staff, we ship in 12 months. With +2 hires ($300k), we ship in 6 months, capturing $1M revenue early."

### 3. Tooling Requests
*   **Shadow IT**: Using free tiers to bypass approval. (Risky).
*   **Consolidation**: Execs love hearing "We are buying Tool X ($10k) but canceling Tool Y and Z ($12k)."

## Go Code Example: Headcount ROI Calculator
This example calculates the financial logic for hiring a new engineer.

```go
package main

import (
	"fmt"
)

type HeadcountRequest struct {
	Role            string
	SalaryCost      float64
	RecruitingFee   float64
	Equipment       float64
	RevenueImpact   float64
}

func (h HeadcountRequest) TotalFirstYearCost() float64 {
	return h.SalaryCost + h.RecruitingFee + h.Equipment
}

func (h HeadcountRequest) ROI() float64 {
	cost := h.TotalFirstYearCost()
	return ((h.RevenueImpact - cost) / cost) * 100
}

func main() {
	req := HeadcountRequest{
		Role:          "Senior Backend Engineer",
		SalaryCost:    150000,
		RecruitingFee: 30000, // 20%
		Equipment:     5000,
		RevenueImpact: 400000, // Value of features shipped
	}

	cost := req.TotalFirstYearCost()
	roi := req.ROI()

	fmt.Println("--- Budget Request ---")
	fmt.Printf("Role: %s\n", req.Role)
	fmt.Printf("Total Cost (Year 1): $%.0f\n", cost)
	fmt.Printf("Expected ROI: %.1f%%\n", roi)
	
	if roi > 0 {
		fmt.Println("Status: ✅ Justifiable Investment")
	} else {
		fmt.Println("Status: ⚠️ Cost Center (Needs Compliance/Stability justification)")
	}
}
```

## Interview Questions

### Q: "We are cutting the budget by 20%. What do you cut?"
**A:**
*   **Protection**: I protect the team (Headcount) first. Layoffs destroy morale and velocity for years.
*   **Infrastructure**: Can we optimize AWS (Spot instances)?
*   **Tools**: Can we downgrade Jira/Slack tiers?
*   **Travel/Events**: Cut the offsite.
*   **Contractors**: Let go of temporary staff first.

### Q: "How do you ask for budget when you can't prove revenue impact (e.g., Refactoring)?"
**A:**
*   **Risk Reduction**: Frame it as insurance. "The cost of an outage is $100k/hour. This refactor prevents that."
*   **Efficiency**: "This tool ($5k) saves each dev 2 hours/week. That is equivalent to hiring 1 new engineer ($150k)."

### Q: "What is 'Sandbagging' a budget?"
**A:**
*   Asking for $200k when you know you need $150k, assuming they will negotiate you down.
*   *Risk*: If found out, you lose credibility. Better to be data-driven and firm.
