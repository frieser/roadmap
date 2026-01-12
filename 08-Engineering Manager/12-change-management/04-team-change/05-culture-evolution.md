# Culture Evolution

## Summary
Culture Evolution is the proactive steering of a team's values and behaviors over time. Culture is not static; it drifts. As teams grow (Start-up -> Scale-up -> Enterprise), the culture *must* change. What worked for 5 people (chaos, speed) kills a team of 50 (needs process, stability).

## Detailed Explanation

### 1. The Dunbar Numbers
*   **~5 people**: Family. Informal. No process needed.
*   **~15 people**: Tribe. Everyone knows everyone. Lightweight process.
*   **~50 people**: Village. Strangers appear. Need documentation and formal HR.
*   **~150 people**: City. Need bureaucracy to function.

### 2. Cultural Debt
*   Like technical debt, cultural shortcuts (e.g., "hiring friends," "ignoring diversity") compound over time.
*   Fixing it requires a "Cultural Refactor" (hard conversations, firing people).

### 3. Preserving the Core
*   When scaling, identify the 2-3 non-negotiable values (e.g., "Customer Obsession").
*   Let everything else evolve (e.g., "We used to deploy on Fridays" -> "Now we have a freeze window").

## Go Code Example: Culture Drift Simulator
This conceptual simulation shows how new hires dilute culture if onboarding isn't strong.

```go
package main

import (
	"fmt"
	"math/rand"
)

type Employee struct {
	CultureScore float64 // 0.0 to 1.0 (Alignment)
}

func SimulateGrowth(initialTeam []Employee, newHires int, onboardingStrength float64) []Employee {
	team := initialTeam
	
	for i := 0; i < newHires; i++ {
		// New hires have random alignment initially
		rawAlignment := rand.Float64()
		
		// Onboarding pulls them closer to 1.0
		adjustedAlignment := rawAlignment + (1.0 - rawAlignment) * onboardingStrength
		
		team = append(team, Employee{CultureScore: adjustedAlignment})
	}
	
	return team
}

func CalculateAvgCulture(team []Employee) float64 {
	sum := 0.0
	for _, e := range team {
		sum += e.CultureScore
	}
	return sum / float64(len(team))
}

func main() {
	founders := []Employee{{1.0}, {1.0}, {1.0}}
	
	// Scenario A: Weak Onboarding (0.2)
	teamA := SimulateGrowth(founders, 50, 0.2)
	fmt.Printf("Scenario A (Weak Onboarding): Avg Alignment %.2f\n", CalculateAvgCulture(teamA))
	
	// Scenario B: Strong Onboarding (0.8)
	teamB := SimulateGrowth(founders, 50, 0.8)
	fmt.Printf("Scenario B (Strong Onboarding): Avg Alignment %.2f\n", CalculateAvgCulture(teamB))
}
```

## Interview Questions

### Q: "How do you maintain 'Startup Culture' as you grow?"
**A:**
*   **You don't**. "Startup Culture" often means "Overwork and Chaos."
*   **You evolve**: Keep the *Speed* (Autonomy) but add *Stability* (Testing).
*   **Divide and Conquer**: Keep teams small (Two-pizza rule) to simulate mini-startups within the enterprise.

### Q: "We hired 20 people and now the culture feels different. Why?"
**A:**
*   **Dilution**: New hires bring their old habits (Amazon culture, Google culture).
*   **Action**: Re-state values explicitly. "Here, we do X." Use the old guard to mentor the new guard.

### Q: "What is a 'Cultural Refactor'?"
**A:**
*   Identifying a toxic trait (e.g., "We blame people for outages") and systematically removing it through training, new rituals (Blameless Post-Mortems), and if necessary, removing the worst offenders.
