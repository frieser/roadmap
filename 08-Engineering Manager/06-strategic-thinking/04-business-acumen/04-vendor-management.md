# Vendor Management

## Summary
Vendor Management is the process of selecting, negotiating with, and managing third-party suppliers (SaaS, Cloud, Agencies). EMs must ensure vendors deliver value, meet security standards, and stay within budget. It moves the relationship from transactional to strategic partnership.

## Detailed Explanation

### 1. Selection Process (RFP)
*   **Build vs. Buy**: First, decide if you really need a vendor (see Technical Strategy).
*   **RFP (Request for Proposal)**: Evaluating multiple vendors against a matrix of requirements (Feature set, Support, Security, Price).
*   **Proof of Concept (PoC)**: Testing the tool in a sandbox before signing.

### 2. Negotiation
*   **Leverage**: Knowing your BATNA (Best Alternative To a Negotiated Agreement).
*   **Terms**: Price is just one variable. Negotiate for:
    *   Better Support SLAs.
    *   Training/Onboarding included.
    *   Flexible seat counts (True-up at end of year).
    *   Exit clauses (Data portability).

### 3. Risk Management
*   **Security Review**: SOC2 compliance, data privacy (GDPR), access controls (SSO).
*   **Vendor Lock-in**: How hard is it to leave? (e.g., heavy dependence on proprietary APIs).

### 4. Relationship Management
*   **QBR (Quarterly Business Review)**: Meeting with the vendor to review usage, roadmap, and issues.
*   **Renewal**: Don't auto-renew. Re-evaluate if the tool is still needed.

## Go Code Example: Vendor Scorecard
This example implements a weighted scoring system to choose between vendors (e.g., Datadog vs. New Relic).

```go
package main

import (
	"fmt"
)

type Vendor struct {
	Name    string
	Scores  map[string]float64 // Criteria -> Score (1-10)
	Price   float64
}

type Criteria struct {
	Name   string
	Weight float64 // Importance (0.1 - 1.0)
}

func EvaluateVendors(vendors []Vendor, criteria []Criteria) {
	fmt.Println("--- Vendor Evaluation Matrix ---")
	
	bestVendor := ""
	maxScore := -1.0

	for _, v := range vendors {
		totalScore := 0.0
		for _, c := range criteria {
			score := v.Scores[c.Name]
			weighted := score * c.Weight
			totalScore += weighted
		}
		
		// Adjust for price (Value for money) - Simplified logic
		// Higher score is better
		
		fmt.Printf("Vendor: %s | Weighted Score: %.2f | Cost: $%.0f\n", v.Name, totalScore, v.Price)
		
		if totalScore > maxScore {
			maxScore = totalScore
			bestVendor = v.Name
		}
	}

	fmt.Printf("\n🏆 Recommendation: %s\n", bestVendor)
}

func main() {
	criteria := []Criteria{
		{"Features", 0.4},
		{"Ease of Use", 0.3},
		{"Support", 0.2},
		{"Security", 0.1},
	}

	vendors := []Vendor{
		{
			Name: "Vendor A",
			Scores: map[string]float64{
				"Features": 9, "Ease of Use": 7, "Support": 8, "Security": 10,
			},
			Price: 50000,
		},
		{
			Name: "Vendor B",
			Scores: map[string]float64{
				"Features": 7, "Ease of Use": 10, "Support": 9, "Security": 8,
			},
			Price: 40000,
		},
	}

	EvaluateVendors(vendors, criteria)
}
```

## Interview Questions

### Q: "How do you handle a critical vendor (e.g., GitHub) going down?"
**A:**
*   **Business Continuity Plan (BCP)**: We should have a plan.
    *   Can we work offline?
    *   Do we have a mirror?
*   **Communication**: Inform stakeholders that velocity is impacted.
*   **SLA Credits**: After the incident, claim the service credits owed from the SLA breach.

### Q: "A dev wants to buy a shiny new tool. What do you do?"
**A:**
*   **Justification**: "What problem does this solve that our current toolstack doesn't?"
*   **Consolidation**: "Can we do this with an existing vendor?" (e.g., use GitLab CI instead of buying CircleCI if we already have GitLab).
*   **Shadow IT**: Warn against putting company code/data on unapproved tools (Security risk).

### Q: "What is the biggest risk in vendor management?"
**A:**
*   **Lock-in**: Being so integrated that they can raise prices 500% and you can't leave.
*   **Mitigation**: Use open standards (e.g., OpenTelemetry) where possible to decouple the data from the vendor.
