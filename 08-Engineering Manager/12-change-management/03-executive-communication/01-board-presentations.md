# Board Presentations

## Summary
Presenting to the Board of Directors is high-stakes communication. Board members are not operational; they are fiduciary. They care about **Risk**, **Strategy**, and **Money**. An EM (or CTO) presenting to the board must translate technical detail into business confidence.

## Detailed Explanation

### 1. The Audience
*   **Investors (VCs)**: Care about growth, valuation, and market fit.
*   **Independent Directors**: Care about governance, compliance, and long-term stability.
*   **CEO**: Wants you to show that Engineering is under control.

### 2. The Content Triangle
1.  **Highlights (Wins)**: "We shipped the AI feature, increasing upsells by 10%."
2.  **Lowlights (Risks)**: "We had a security incident (mitigated). We are delayed on Project X."
3.  **The Ask**: "We need to hire 3 engineers to hit the Q4 goal."

### 3. Anti-Patterns
*   **Getting Weedsy**: Explaining *how* Kubernetes works. (They don't care).
*   **Surprises**: Bad news must be shared with the CEO *before* the meeting.
*   **Hiding Bad News**: This destroys trust. "Bad news travels fast; good news travels slow."

## Go Code Example: Board Metric Dashboard
This example generates a high-level summary struct suitable for a slide deck, filtering out low-level noise.

```go
package main

import (
	"fmt"
)

type BoardMetric struct {
	Name      string
	Value     string
	Trend     string // "Up", "Down", "Flat"
	Status    string // "Green", "Yellow", "Red"
	Highlight string // The "So What?" explanation
}

func GenerateBoardDeck() {
	metrics := []BoardMetric{
		{
			Name: "System Uptime", Value: "99.98%", Trend: "Flat", Status: "Green",
			Highlight: "Stable platform supporting record traffic.",
		},
		{
			Name: "R&D Hiring", Value: "85% to Goal", Trend: "Down", Status: "Yellow",
			Highlight: "Sourcing for Senior Backend roles is slower than expected.",
		},
		{
			Name: "Cloud Costs", Value: "$50k/mo", Trend: "Up", Status: "Red",
			Highlight: "Spike due to unoptimized AI models. Remediation plan in place.",
		},
	}

	fmt.Println("--- Q3 Engineering Board Update ---")
	for _, m := range metrics {
		icon := "✅"
		if m.Status == "Yellow" { icon = "⚠️" }
		if m.Status == "Red" { icon = "🚨" }

		fmt.Printf("%s **%s**: %s (%s)\n", icon, m.Name, m.Value, m.Trend)
		fmt.Printf("   -> Insight: %s\n\n", m.Highlight)
	}
}

func main() {
	GenerateBoardDeck()
}
```

## Interview Questions

### Q: "How do you explain a major outage to the Board?"
**A:**
*   **Own it**: "We were down for 2 hours."
*   **Impact**: "It cost us ~$20k in revenue."
*   **Root Cause (High Level)**: "Capacity limits were hit."
*   **Solution**: "We have implemented auto-scaling to prevent recurrence."
*   **Tone**: Confident, factual, not defensive.

### Q: "The Board asks why development is 'so slow'. What do you say?"
**A:**
*   **Don't complain** about complexity.
*   **Contextualize**: "We are investing 30% of capacity in paying down debt from the initial MVP. This will allow us to go faster in Q2."
*   **Metrics**: Show Cycle Time trends. "Actually, our deployment frequency is up 20% year-over-year."

### Q: "What is the goal of a Board Meeting?"
**A:**
*   To keep the board **informed** so they can provide **governance** and **resources**. It is not a brainstorming session (usually).
