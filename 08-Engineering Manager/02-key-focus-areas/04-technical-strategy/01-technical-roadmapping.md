# Technical Roadmapping

## Summary
Technical Roadmapping is the process of creating a strategic plan that aligns technical execution with business goals. It visualizes the evolution of the software architecture, infrastructure, and toolset over time (typically 6-18 months). A good roadmap manages dependencies, anticipates future needs, and communicates priorities to stakeholders.

## Detailed Explanation
A technical roadmap is different from a product roadmap. While the product roadmap focuses on user features (what), the technical roadmap focuses on the enabling technology (how).

### Core Elements
1.  **Strategic Alignment**: Every item on the roadmap should support a business objective (e.g., "Refactor payment service" -> "Support new international markets").
2.  **Horizons**:
    *   **Now (1-3 months)**: High confidence, detailed tasks.
    *   **Next (3-6 months)**: Broad goals, major milestones.
    *   **Later (6-12+ months)**: Strategic bets, aspirational targets.
3.  **Themes**: Grouping work into categories like "Scale," "Security," "Developer Productivity," or "Stability."

### The Process
1.  **Gap Analysis**: Where are we now vs. where do we need to be?
2.  **Prioritization**: Using frameworks like RICE (Reach, Impact, Confidence, Effort) or MoSCoW to rank initiatives.
3.  **Stakeholder Communication**: Translating technical jargon into business value/risk reduction for non-technical leadership.

### Common Pitfalls
- **The "Wishlist"**: A roadmap that is just a list of cool tech with no business justification.
- **Static Artifacts**: Roadmaps that are created once and never updated. They must be living documents.
- **Ignoring Maintenance**: Failing to schedule upgrades and debt paydown, leading to a "feature factory" that eventually stalls.

## Go Code Example
This example models a Roadmap Item and a basic Prioritization Engine using the RICE scoring method.

```go
package main

import (
	"fmt"
	"sort"
)

// Initiative represents a technical project on the roadmap
type Initiative struct {
	Name        string
	Reach       float64 // How many people/systems will this affect?
	Impact      float64 // (3 = massive, 2 = high, 1 = medium, 0.5 = low, 0.25 = minimal)
	Confidence  float64 // (1.0 = high, 0.8 = medium, 0.5 = low)
	Effort      float64 // Person-months
}

// RICE Score = (Reach * Impact * Confidence) / Effort
func (i Initiative) CalculateScore() float64 {
	if i.Effort == 0 {
		return 0
	}
	return (i.Reach * i.Impact * i.Confidence) / i.Effort
}

// RoadmapManager handles prioritization
type RoadmapManager struct {
	Backlog []Initiative
}

func (rm *RoadmapManager) Prioritize() {
	sort.Slice(rm.Backlog, func(i, j int) bool {
		return rm.Backlog[i].CalculateScore() > rm.Backlog[j].CalculateScore()
	})
}

func main() {
	manager := RoadmapManager{
		Backlog: []Initiative{
			{
				Name:       "Migrate to Kubernetes",
				Reach:      1000, // Affects all devs
				Impact:     2,    // High impact on scalability
				Confidence: 0.8,  // Pretty sure it will help
				Effort:     6,    // 6 person-months
			},
			{
				Name:       "Fix Flaky Tests",
				Reach:      50,   // Affects active committers
				Impact:     1,    // Medium impact
				Confidence: 1.0,  // Certain outcome
				Effort:     1,    // 1 person-month
			},
			{
				Name:       "Rewrite in Rust",
				Reach:      5,    // Affects small team
				Impact:     0.5,  // Low business impact currently
				Confidence: 0.5,  // Unsure of payoff
				Effort:     12,   // High effort
			},
		},
	}

	manager.Prioritize()

	fmt.Println("--- Technical Roadmap Priorities (RICE Score) ---")
	for i, task := range manager.Backlog {
		fmt.Printf("%d. %s (Score: %.2f)\n", i+1, task.Name, task.CalculateScore())
	}
}
```

## Interview Questions
1.  **How do you convince a product manager to prioritize technical debt on the roadmap?**
    *   *Focus*: Translating "debt" into "risk" (slow delivery, potential outages), negotiating a % capacity allocation (e.g., 20%).
2.  **Describe your process for creating a technical roadmap for the next 6 months.**
    *   *Focus*: Gathering input from the team, aligning with business goals, estimating effort, communicating to stakeholders.
3.  **How do you handle roadmap changes when urgent business requests come in?**
    *   *Focus*: Assessing trade-offs, communicating the impact of the delay ("If we do X, Y will slip"), avoiding "squeezing it in."
4.  **What frameworks do you use for prioritization?**
    *   *Focus*: RICE, MoSCoW, Cost of Delay, ROI.
