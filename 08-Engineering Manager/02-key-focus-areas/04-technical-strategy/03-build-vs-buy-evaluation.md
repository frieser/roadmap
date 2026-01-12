# Build vs. Buy Evaluation

## Summary
Build vs. Buy is the strategic evaluation of whether to develop a solution internally ("Build") or license/purchase an existing product ("Buy"). The decision hinges on whether the capability is a core competitive differentiator (Build) or a commodity utility (Buy), balancing long-term TCO (Total Cost of Ownership) against speed to market.

## Detailed Explanation
Engineering teams often have a bias to build. Managers must counter this with objective analysis.

### The Core Question
**"Is this core to our business?"**
- **Yes (Core)**: Build. Owning the IP gives a competitive advantage (e.g., Google's search algorithm, Netflix's recommendation engine).
- **No (Context)**: Buy. It's a solved problem that supports the business but doesn't differentiate it (e.g., Payroll, CRM, Authentication, Logging).

### Evaluation Criteria
1.  **Cost**:
    *   *Buy*: License fees, integration costs.
    *   *Build*: Salary (dev + maintenance forever), infrastructure, opportunity cost (what are we *not* building?).
2.  **Time to Market**: Buying is almost always faster.
3.  **Control**: Building offers 100% control; Buying relies on a vendor's roadmap.
4.  **Maintenance**: Building creates a permanent maintenance liability.

### "Rent" (SaaS)
Modern "Buy" is usually "Rent" (SaaS). This shifts CapEx to OpEx and offloads operational burden.

## Go Code Example
This example calculates the Total Cost of Ownership (TCO) for a Build vs. Buy scenario over a 3-year period to assist in decision-making.

```go
package main

import (
	"fmt"
)

type CostConfig struct {
	InitialCost     float64 // Setup or Dev time
	MonthlyCost     float64 // Subscription or Maintenance hours
	OpportunityCost float64 // Value lost by building this instead of product features
}

type TCOResult struct {
	Year1 float64
	Year3 float64
}

func CalculateTCO(name string, cfg CostConfig) TCOResult {
	y1 := cfg.InitialCost + (cfg.MonthlyCost * 12) + cfg.OpportunityCost
	y3 := cfg.InitialCost + (cfg.MonthlyCost * 36) + cfg.OpportunityCost
	return TCOResult{Year1: y1, Year3: y3}
}

func main() {
	// Scenario: Authentication System
	
	// Option A: Build (OAuth2 server)
	// - 3 devs for 2 months to build ($100k)
	// - 10 hours/month maintenance ($1k)
	// - High opportunity cost ($50k)
	buildCfg := CostConfig{
		InitialCost:     100000,
		MonthlyCost:     1000,
		OpportunityCost: 50000,
	}

	// Option B: Buy (Auth0/Okta)
	// - Integration time ($5k)
	// - $2000/month subscription
	// - Zero opportunity cost
	buyCfg := CostConfig{
		InitialCost:     5000,
		MonthlyCost:     2000,
		OpportunityCost: 0,
	}

	buildRes := CalculateTCO("Build", buildCfg)
	buyRes := CalculateTCO("Buy", buyCfg)

	fmt.Println("--- Build vs Buy TCO Analysis (Authentication) ---")
	fmt.Printf("BUILD | Year 1: $%.0f | 3 Year: $%.0f\n", buildRes.Year1, buildRes.Year3)
	fmt.Printf("BUY   | Year 1: $%.0f  | 3 Year: $%.0f\n", buyRes.Year1, buyRes.Year3)

	if buyRes.Year3 < buildRes.Year3 {
		fmt.Println("\nRecommendation: BUY (Lower 3-year TCO and faster time-to-market)")
	} else {
		fmt.Println("\nRecommendation: BUILD (Long term savings justify initial effort)")
	}
}
```

## Interview Questions
1.  **When would you advocate for building a tool internally that already exists on the market?**
    *   *Focus*: Unique business requirements, scale costs of vendor prohibitive, vendor lock-in risks, core competency.
2.  **How do you calculate the "maintenance tax" of a home-grown solution?**
    *   *Focus*: Upgrades, bug fixes, onboarding new devs, security patches (often estimated at 20% of initial build cost per year).
3.  **A team member wants to build a custom message queue because "Kafka is too complex." How do you handle this?**
    *   *Focus*: Identify the root cause (complexity), suggest managed services (AWS MSK) vs. raw install, challenge the "Not Invented Here" (NIH) syndrome.
4.  **What is the "Opportunity Cost" in a build vs. buy decision?**
    *   *Focus*: The revenue-generating features the team *could* have built if they weren't building utility software.
