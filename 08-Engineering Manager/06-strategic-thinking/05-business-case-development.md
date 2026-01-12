# Business Case Development

## Summary
Business Case Development is the skill of justifying a proposed project or investment to stakeholders. It answers the question: "Why should we spend money and time on this instead of something else?" For EMs, this often involves advocating for technical initiatives (like refactoring, migrations, or new tools) by translating them into business value.

## Detailed Explanation

### 1. The Structure of a Business Case
A strong business case typically follows this narrative arc:
*   **The Problem**: "Our current billing system is slow and error-prone." (Pain).
*   **The Opportunity**: "If we fix it, we can launch pricing experiments 10x faster." (Gain).
*   **The Solution**: "Migrate to Stripe." (Method).
*   **The Cost**: "$50k implementation + $2k/month." (Investment).
*   **The ROI**: "We save $100k/year in manual reconciliation." (Return).

### 2. Translating Tech to Business
Stakeholders don't care about "Technical Debt" or "Kubernetes." They care about:
*   **Revenue**: Will this help us sell more?
*   **Cost**: Will this save us money?
*   **Risk**: Will this prevent a lawsuit or outage?
*   *Example*: Don't say "Upgrade React." Say "Reduce page load time by 50% to improve SEO rankings and conversion."

### 3. Handling Objections
*   **"It's too expensive."** -> Show the *Cost of Inaction* (e.g., "If we don't do this, we will likely crash on Black Friday").
*   **"Can we do it later?"** -> Explain the compound interest of debt.

## Go Code Example: Proposal Scorer
This example scores different project proposals based on a weighted matrix of Strategic Fit, Financial Impact, and Risk.

```go
package main

import (
	"fmt"
)

type Proposal struct {
	Name          string
	StrategicFit  int // 1-5: Aligns with company goals?
	RevenueImpact int // 1-5: Makes money?
	CostSavings   int // 1-5: Saves money?
	RiskMitigation int // 1-5: Prevents disaster?
	Effort        int // 1-5: How hard is it? (Lower is better for score)
}

func (p Proposal) CalculateScore() float64 {
	// Weighted Formula
	// Strategy (30%) + Revenue (30%) + Savings (20%) + Risk (20%)
	// Divided by Effort penalty
	
	valueScore := (float64(p.StrategicFit) * 0.3) +
		(float64(p.RevenueImpact) * 0.3) +
		(float64(p.CostSavings) * 0.2) +
		(float64(p.RiskMitigation) * 0.2)
	
	// Normalize effort: 1=1.0, 5=0.2 (High effort reduces score)
	effortFactor := 1.0 / float64(p.Effort) * 2.5 
	
	return valueScore * effortFactor
}

func main() {
	proposals := []Proposal{
		{
			Name: "Rewrite Legacy Monolith", 
			StrategicFit: 5, RevenueImpact: 2, CostSavings: 3, RiskMitigation: 4, 
			Effort: 5, // Huge effort
		},
		{
			Name: "Add Apple Pay Support", 
			StrategicFit: 4, RevenueImpact: 5, CostSavings: 1, RiskMitigation: 1, 
			Effort: 2, // Low effort, High revenue
		},
	}

	fmt.Println("--- Business Case Priority ---")
	for _, p := range proposals {
		fmt.Printf("Project: %s | Score: %.2f\n", p.Name, p.CalculateScore())
	}
}
```

## Interview Questions

### Q: "How do you convince a non-technical CEO to fund a major refactor?"
**A:**
*   **Focus on Risk and Velocity**: "Currently, every feature takes 2 weeks to build because the code is brittle. If we refactor, we can cut that to 3 days. Also, we are at high risk of a security breach due to outdated libraries."
*   **Analogy**: "It's like a kitchen. If we don't wash the dishes (refactor), we can't cook the next meal (ship features)."

### Q: "What are the key components of a 'One-Pager' proposal?"
**A:**
*   **Problem Statement**: 1-2 sentences clearly defining the gap.
*   **Proposed Solution**: High-level approach.
*   **Success Metrics**: How will we know it worked?
*   **Risks/Trade-offs**: What could go wrong?

### Q: "Describe a time your business case was rejected. What did you do?"
**A:**
*   **Listen**: Understood *why* (usually "not right now" or "budget constraints").
*   **Pivot**: Broke the project down into smaller, cheaper chunks.
*   **Resubmit**: Proposed a "Phase 1" with lower risk and clearer immediate ROI.
