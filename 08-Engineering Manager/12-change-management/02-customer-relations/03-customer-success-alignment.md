# Customer Success Alignment

## Summary
Customer Success (CS) is distinct from Support. Support fixes problems (Reactive); Success helps customers achieve value (Proactive). For EMs, aligning with CS helps identify "Expansion" opportunities (Upsell) and reduces "Churn." CS Managers (CSMs) are your best source of intel on how power users actually use the system.

## Detailed Explanation

### 1. The Feedback Loop
*   CSMs talk to the biggest customers daily.
*   **EM Role**: Set up a monthly sync with CS Leadership. "What are the top 3 friction points preventing renewal?"
*   **Data**: Use this qualitative data to adjust the roadmap.

### 2. Feature Adoption
*   Engineering ships a feature -> CS trains the customer -> Customer uses it.
*   If Adoption is low, is it a *Sales* problem (didn't sell it), a *Product* problem (wrong feature), or an *Engineering* problem (buggy/slow)?
*   Alignment ensures we debug the "Adoption Funnel" together.

### 3. Technical Account Managers (TAMs)
*   For Enterprise software, TAMs are hybrid Engineer-Consultants.
*   They need direct access to Engineering for architecture reviews. Treat them as extended team members.

## Go Code Example: Churn Prediction Signal
This example aggregates usage signals (from Engineering) to alert CS about at-risk customers.

```go
package main

import (
	"fmt"
)

type Customer struct {
	Name            string
	LoginsLast30Days int
	TicketsLast30Days int
	ContractValue   int
}

func PredictRisk(c Customer) string {
	// Rule 1: No login = High Risk
	if c.LoginsLast30Days == 0 {
		return "🔴 HIGH RISK (Abandoned)"
	}
	
	// Rule 2: Low login + High Tickets = Frustrated
	if c.LoginsLast30Days < 5 && c.TicketsLast30Days > 3 {
		return "🔴 HIGH RISK (Frustrated)"
	}

	// Rule 3: High Value + Dropping usage
	// (Simplified logic)
	
	return "🟢 Healthy"
}

func main() {
	customers := []Customer{
		{"BigCorp", 150, 2, 50000},
		{"GhostLLC", 0, 0, 10000},
		{"AngryInc", 4, 10, 20000},
	}

	fmt.Println("--- Customer Health Monitor ---")
	for _, c := range customers {
		status := PredictRisk(c)
		fmt.Printf("%s: %s\n", c.Name, status)
		
		if status != "🟢 Healthy" {
			fmt.Println("   -> Action: Alert CSM immediately.")
		}
	}
}
```

## Interview Questions

### Q: "How can Engineering help Customer Success hit their retention goals?"
**A:**
*   **Stability**: Uptime is the #1 retention feature.
*   **Observability**: Give CSMs a dashboard where they can see *if* their customer is using the tool, so they can intervene early.
*   **Responsiveness**: Prioritize bugs affecting high-value renewal accounts.

### Q: "A CSM promises a feature to save a renewal without asking Engineering. What do you do?"
**A:**
*   **Short term**: Assess impact. Can we do it? If yes, do it to save the account (Strategic debt).
*   **Long term**: "Never again." Establish a process. "Sales/CS cannot commit roadmap without Engineering approval."
*   **Education**: Explain *why* interruptions hurt everyone else.

### Q: "What is the difference between Customer Support and Customer Success?"
**A:**
*   **Support**: Break/Fix. Reactive. "My car is broken."
*   **Success**: Value/Growth. Proactive. "How do I drive to the beach?"
