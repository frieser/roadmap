# Resource Forecasting

## Summary
Resource Forecasting is the art of predicting the team size and skills needed to meet future business goals. It bridges the gap between the Product Roadmap ("We want to build X") and Hiring ("We need Y engineers"). Effective forecasting prevents burnout (under-staffing) and idleness (over-staffing).

## Detailed Explanation

### 1. Capacity Planning
*   **Supply**: How many engineering hours/points do we have? (Team Size * Velocity * Availability).
*   **Demand**: How big are the projects on the roadmap? (T-Shirt sizing or estimation).
*   **Gap**: Demand - Supply = Hiring Need.

### 2. The Myth of the Man-Month
> "Adding manpower to a late software project makes it later." — Fred Brooks.
*   New hires have negative productivity initially (onboarding takes time from seniors).
*   Forecasting must account for ramp-up time (typically 3-6 months for full productivity).

### 3. Ratios and Heuristics
*   **Manager to IC Ratio**: Typically 1:6 to 1:10.
*   **QA to Dev Ratio**: Varies (1:3 to 0:Infinite if fully automated).
*   **Product/Design to Dev**: Often 1 PM : 1 Designer : 5-8 Devs.

## Go Code Example: Capacity Calculator
This example calculates whether a team has enough capacity to meet a deadline, accounting for ramp-up time of new hires.

```go
package main

import (
	"fmt"
)

type Engineer struct {
	Name           string
	Productivity   float64 // 1.0 = Fully productive, 0.5 = New hire
}

type Team struct {
	Members []Engineer
}

func (t Team) WeeklyCapacity() float64 {
	total := 0.0
	for _, e := range t.Members {
		total += e.Productivity
	}
	return total // In "Engineer-Weeks"
}

func main() {
	// Current Team
	team := Team{
		Members: []Engineer{
			{"Alice", 1.0},
			{"Bob", 1.0},
			{"Charlie", 1.0},
		},
	}

	projectEstimateWeeks := 50.0 // 1 engineer would take 50 weeks
	deadlineWeeks := 12.0

	currentCap := team.WeeklyCapacity() // 3.0
	weeksNeeded := projectEstimateWeeks / currentCap

	fmt.Printf("Current Capacity: %.1f engineer-weeks/week\n", currentCap)
	fmt.Printf("Time to finish: %.1f weeks\n", weeksNeeded)

	if weeksNeeded > deadlineWeeks {
		fmt.Println("⚠️  Project is AT RISK.")
		
		// What if we hire 2 people?
		// New hires are 0% productive week 1, 50% productive week 4...
		// Simplified: assume 0.5 avg for first quarter
		newHires := []Engineer{{"NewHire1", 0.5}, {"NewHire2", 0.5}}
		team.Members = append(team.Members, newHires...)
		
		newCap := team.WeeklyCapacity() // 4.0
		newTime := projectEstimateWeeks / newCap
		
		fmt.Printf("With 2 new hires (ramping up): %.1f weeks\n", newTime)
	} else {
		fmt.Println("✅ Project is on track.")
	}
}
```

## Interview Questions

### Q: "Product wants to double the roadmap next quarter. Can we just double the team?"
**A:**
*   **No**. Doubling the team breaks communication structures (Metcalfe's Law) and culture.
*   **Brooks' Law**: New hires will slow down the current team for training.
*   **Counter-proposal**: Scale gradually (e.g., 20% growth) or prioritize the roadmap ruthlessly.

### Q: "How do you estimate capacity when you have unplanned work (bugs/incidents)?"
**A:**
*   **Load Factor**: I never plan for 100% capacity. I plan for ~70-80%.
*   **The Rest**: 20% is reserved for "Keep the Lights On" (KTLO), bugs, and meetings. If we track velocity, this is baked in automatically.

### Q: "How do you forecast for a project that is very vague?"
**A:**
*   **Cone of Uncertainty**: Give a range, not a number (e.g., "3-6 months").
*   **Discovery Spike**: Assign 1 engineer for 1 week to investigate and clarify the scope before committing resources.
