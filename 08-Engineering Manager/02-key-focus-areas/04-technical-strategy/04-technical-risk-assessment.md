# Technical Risk Assessment

## Summary
Technical Risk Assessment is the systematic process of identifying, analyzing, and mitigating risks inherent in software development and infrastructure. It shifts engineering from a reactive posture ("fixing fires") to a proactive one ("preventing fires"). Risks can be architectural, operational, security-related, or tied to dependencies.

## Detailed Explanation
Risks are future uncertainties that, if realized, have a negative impact. The goal is not to eliminate all risk (which stops innovation) but to manage it.

### Risk Management Cycle
1.  **Identify**: Brainstorm potential failures (e.g., "What if the DB goes down?", "What if this API is deprecated?").
2.  **Assess**:
    *   **Likelihood**: Probability of occurrence (High/Medium/Low).
    *   **Impact**: Severity of damage (High/Medium/Low).
3.  **Plan Response**:
    *   **Avoid**: Change plans to eliminate the risk.
    *   **Mitigate**: Reduce likelihood or impact (e.g., add redundancy).
    *   **Transfer**: Move risk to a third party (e.g., insurance, managed services).
    *   **Accept**: Acknowledge it and do nothing (for low risks).
4.  **Monitor**: Regularly review risks.

### Categories of Technical Risk
- **Single Point of Failure (SPOF)**: Components without redundancy.
- **Bus Factor**: Only one person understands the code.
- **Legacy Decay**: Older systems becoming unmaintainable.
- **Scalability Limits**: Systems that will break under expected future load.

## Go Code Example
This example implements a "Risk Matrix" calculator that ingests a list of identified technical risks and produces a prioritized heatmap/report based on the standard `Risk Score = Probability * Impact` formula.

```go
package main

import (
	"fmt"
	"sort"
)

// Risk levels
const (
	Low    = 1
	Medium = 2
	High   = 3
)

type TechnicalRisk struct {
	Description string
	Probability int // 1-3
	Impact      int // 1-3
}

func (r TechnicalRisk) Score() int {
	return r.Probability * r.Impact
}

func (r TechnicalRisk) SeverityLabel() string {
	score := r.Score()
	if score >= 6 {
		return "CRITICAL"
	} else if score >= 3 {
		return "MAJOR"
	}
	return "MINOR"
}

func main() {
	risks := []TechnicalRisk{
		{"Lead Engineer leaves (Bus Factor)", Low, High},           // 1 * 3 = 3
		{"Database disk fills up", Medium, High},                   // 2 * 3 = 6
		{"Third-party API rate limiting", High, Medium},            // 3 * 2 = 6
		{"CSS minor display bug", High, Low},                       // 3 * 1 = 3
		{"Experimental library deprecation", Medium, Medium},       // 2 * 2 = 4
	}

	// Sort by Risk Score (Descending)
	sort.Slice(risks, func(i, j int) bool {
		return risks[i].Score() > risks[j].Score()
	})

	fmt.Println("--- Technical Risk Register ---")
	for _, r := range risks {
		fmt.Printf("[%s] Score: %d | %s\n", r.SeverityLabel(), r.Score(), r.Description)
	}

	fmt.Println("\n--- Mitigation Plan ---")
	for _, r := range risks {
		if r.Score() >= 6 {
			fmt.Printf("IMMEDIATE ACTION REQUIRED: %s\n", r.Description)
		}
	}
}
```

## Interview Questions
1.  **How do you identify "Single Points of Failure" in a system?**
    *   *Focus*: Architecture diagrams, "What if this node dies?" exercises, bus factor analysis.
2.  **How do you communicate technical risk to non-technical stakeholders?**
    *   *Focus*: Translate to business impact (revenue loss, reputation damage, project delay) rather than technical jargon.
3.  **What is "Bus Factor" and how do you mitigate it?**
    *   *Focus*: Documentation, pair programming, code ownership rotation, tech talks.
4.  **Describe a time you proactively identified a risk and prevented an outage.**
    *   *Focus*: Foresight, monitoring trends (e.g., disk usage growing), load testing.
