# ROI Analysis

## Summary
ROI (Return on Investment) Analysis is the quantitative side of strategic thinking. It involves calculating the payback of engineering efforts. EMs use ROI to prioritize the roadmap, choosing projects that deliver the highest value for the lowest cost.

## Detailed Explanation

### 1. The ROI Formula
$$ROI = \frac{\text{Net Benefit}}{\text{Cost}} \times 100$$
*   **Net Benefit**: (Revenue Generated + Costs Saved) - Implementation Cost.
*   **Cost**: Salary (Time) + Infrastructure + Tools.

### 2. Hard ROI vs. Soft ROI
*   **Hard ROI**: Direct financial impact. "Stopping this AWS leak saves $5k/month." (Easy to prove).
*   **Soft ROI**: Indirect impact. "Better developer experience improves morale." (Hard to prove).
*   *Strategy*: Always try to convert Soft to Hard. "Improved morale" -> "Reduced attrition" -> "Saved $30k recruiting fees."

### 3. Opportunity Cost
*   "If we do Project A, we cannot do Project B."
*   ROI analysis must consider what you *give up*. If Project A has 10% ROI and Project B has 50% ROI, doing Project A actually loses you money (relative to the optimal choice).

### 4. Break-even Analysis
*   "How long until this pays for itself?" (Payback Period).
*   Short payback periods (< 6 months) are easier to approve than long ones (> 2 years).

## Go Code Example: ROI Calculator with Time Value
This example compares two projects, factoring in the time horizon.

```go
package main

import (
	"fmt"
)

type Investment struct {
	Name        string
	InitialCost float64
	MonthlyGain float64
}

func (i Investment) CalculateROI(months int) float64 {
	totalGain := i.MonthlyGain * float64(months)
	netBenefit := totalGain - i.InitialCost
	return (netBenefit / i.InitialCost) * 100
}

func (i Investment) BreakEvenMonth() float64 {
	return i.InitialCost / i.MonthlyGain
}

func main() {
	// Scenario: Automating a manual report
	// Cost: 1 week of dev time ($3000)
	// Gain: Saves 2 hours/week of PM time ($100/week = $400/month)
	automation := Investment{
		Name:        "Automate Weekly Report",
		InitialCost: 3000,
		MonthlyGain: 400,
	}

	fmt.Printf("--- Project: %s ---\n", automation.Name)
	fmt.Printf("Break-even: %.1f months\n", automation.BreakEvenMonth())
	fmt.Printf("1-Year ROI: %.1f%%\n", automation.CalculateROI(12))
	fmt.Printf("3-Year ROI: %.1f%%\n", automation.CalculateROI(36))
	
	if automation.BreakEvenMonth() > 12 {
		fmt.Println("⚠️  Long payback period. Re-evaluate priority.")
	} else {
		fmt.Println("✅ High value quick win.")
	}
}
```

## Interview Questions

### Q: "How do you calculate ROI for a platform migration (e.g., Mono to Microservices)?"
**A:**
*   **Costs**: Massive initial dev effort, retraining, new infra complexity.
*   **Benefits**: Increased velocity (independent deployments), independent scaling (cloud savings).
*   **Calculation**: Estimate the "Time saved per feature" post-migration. If we ship 100 features/year and save 2 days per feature, that's 200 days saved/year.

### Q: "Is ROI the only factor in prioritization?"
**A:**
*   **No**.
*   **Compliance/Legal**: ROI might be negative, but we go to jail if we don't do it (GDPR).
*   **Strategic Bets**: Losing money now to capture a market later (e.g., Uber subsidizing rides).

### Q: "How do you measure the ROI of fixing a bug?"
**A:**
*   **Frequency x Impact**: How many users hit it? x How much money do they lose?
*   If a bug blocks checkout for 1% of users, and we make $1M/day, the bug costs $10k/day. Fixing it (1 day work) has an ROI of infinity.
