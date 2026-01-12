# Team Health Metrics

## Summary
Team Health Metrics measure the human side of engineering: morale, retention, engagement, and sustainability. While technical metrics measure the *machine*, health metrics measure the *operators*. Ignoring these leads to burnout and attrition, which are the most expensive costs in engineering.

## Detailed Explanation

### 1. eNPS (Employee Net Promoter Score)
*   **Question**: "On a scale of 0-10, how likely are you to recommend this team/company as a place to work?"
*   **Formula**: `% Promoters (9-10)` - `% Detractors (0-6)`.
*   **Usage**: A leading indicator of attrition.

### 2. Attrition / Retention Rate
*   **Metric**: What percentage of the team leaves annually?
*   **Regrettable vs. Non-Regrettable**: Losing a top performer (Regrettable) is different from losing a poor performer (Non-Regrettable).

### 3. Bus Factor
*   **Definition**: The minimum number of team members that have to "get hit by a bus" before the project cannot proceed.
*   **Goal**: Bus Factor > 1 for all critical components.
*   **Mitigation**: Documentation, Pair Programming, Rotation.

### 4. Meeting Load
*   **Metric**: Hours spent in meetings vs. "Maker Time."
*   **Goal**: Engineers typically need 4-hour blocks of uninterrupted time to be productive.

## Go Code Example: eNPS Calculator
This example calculates the eNPS score from a survey of raw ratings.

```go
package main

import (
	"fmt"
)

type SurveyResponse struct {
	Rating int // 0-10
}

func CalculateENPS(responses []SurveyResponse) (float64, string) {
	if len(responses) == 0 {
		return 0, "N/A"
	}

	promoters := 0
	detractors := 0
	neutrals := 0

	for _, r := range responses {
		if r.Rating >= 9 {
			promoters++
		} else if r.Rating <= 6 {
			detractors++
		} else {
			neutrals++
		}
	}

	total := float64(len(responses))
	pctPromoters := (float64(promoters) / total) * 100
	pctDetractors := (float64(detractors) / total) * 100
	
	enps := pctPromoters - pctDetractors
	
	// Classification
	classification := "Excellent"
	if enps < 0 {
		classification = "Critical Issues"
	} else if enps < 30 {
		classification = "Good"
	}

	return enps, classification
}

func main() {
	// 0-6: Detractor, 7-8: Passive, 9-10: Promoter
	responses := []SurveyResponse{
		{10}, {9}, {10}, // 3 Promoters
		{8}, {7},        // 2 Passives
		{6}, {2},        // 2 Detractors
	}
	
	// Calculation:
	// Total = 7
	// Promoters = 3 (42.8%)
	// Detractors = 2 (28.5%)
	// eNPS = 42.8 - 28.5 = 14.3
	
	score, status := CalculateENPS(responses)
	fmt.Printf("eNPS Score: %.1f (%s)\n", score, status)
	
	if score < 10 {
		fmt.Println("Action: Conduct 'Stay Interviews' to understand dissatisfaction.")
	}
}
```

## Interview Questions

### Q: "How do you identify burnout before an employee quits?"
**A:**
*   **Behavioral changes**: Withdrawal from social channels, cameras off in meetings, cynicism.
*   **Metric changes**: Sharp drop in velocity, increase in PR cycle time (procrastination), working odd hours (weekends/late nights).

### Q: "Your team has a Bus Factor of 1 on the payment service. What do you do?"
**A:**
*   **Immediate**: Record a "knowledge transfer" session where the expert explains the code.
*   **Short-term**: Assign a "shadow" to the expert for the next feature on that service.
*   **Long-term**: Rotate on-call duties so others are forced to learn the system.

### Q: "How do you handle a low eNPS score?"
**A:**
*   **Acknowledge it**: Don't hide the result. Tell the team, "We heard you."
*   **Focus Groups**: Dig into the qualitative data. Is it pay? Tooling? Management?
*   **One Action**: Pick one thing to fix immediately to build trust.
