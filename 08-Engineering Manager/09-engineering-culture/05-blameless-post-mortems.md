## Summary
Blameless Post-Mortems (or Incident Retrospectives) are structured analyses of incidents focused on process and system failures rather than human error. The goal is to uncover the systemic root causes to prevent recurrence, fostering a culture where engineers feel safe reporting issues immediately.

## Detailed Explanation
When an incident occurs, "who" caused it is irrelevant; "how" the system allowed it to happen is everything. Humans are fallible; systems should be resilient.

### The Core Philosophy
*   **You can't fire your way to reliability**: Blaming individuals leads to covering up mistakes.
*   **Root Cause Analysis**: Using techniques like "The 5 Whys" to dig deep.
*   **Actionable Outcomes**: Every post-mortem must yield specific tasks (Jira tickets) to improve the system.

### The Process
1.  **Detect & Resolve**: Fix the immediate fire.
2.  **Document**: Timeline of events (Who, What, When).
3.  **Analyze**: Why did it happen? Why wasn't it detected sooner?
4.  **Remediate**: Create action items.
5.  **Share**: Publish the report to the wider org.

## Go Code Example
Modeling an `IncidentReport` and a "5 Whys" analyzer structure.

```go
package main

import (
	"fmt"
	"strings"
)

type RootCauseAnalysis struct {
	ProblemStatement string
	FiveWhys         []string
}

type IncidentReport struct {
	ID           string
	Title        string
	Severity     string // SEV1, SEV2, etc.
	Analysis     RootCauseAnalysis
	ActionItems  []string
}

// ConductAnalysis mimics the iterative questioning process
func (r *RootCauseAnalysis) AddWhy(reason string) {
	if len(r.FiveWhys) < 5 {
		r.FiveWhys = append(r.FiveWhys, reason)
	}
}

func (i IncidentReport) GenerateMarkdown() string {
	var sb strings.Builder
	sb.WriteString(fmt.Sprintf("# Incident: %s (%s)\n", i.Title, i.Severity))
	sb.WriteString("## Root Cause Analysis (5 Whys)\n")
	for idx, why := range i.Analysis.FiveWhys {
		sb.WriteString(fmt.Sprintf("%d. %s\n", idx+1, why))
	}
	sb.WriteString("\n## Action Items\n")
	for _, item := range i.ActionItems {
		sb.WriteString(fmt.Sprintf("- [ ] %s\n", item))
	}
	return sb.String()
}

func main() {
	report := IncidentReport{
		ID:       "INC-2023-101",
		Title:    "Production Database Latency Spike",
		Severity: "SEV1",
		ActionItems: []string{
			"Add rate limiting to API gateway",
			"Cache expensive query results in Redis",
		},
	}

	// Simulating the 5 Whys session
	report.Analysis.ProblemStatement = "Database CPU spiked to 100%"
	report.Analysis.AddWhy("A complex query was run frequently.")
	report.Analysis.AddWhy("The dashboard refreshed every 5 seconds.")
	report.Analysis.AddWhy("No caching layer was implemented for this endpoint.")
	report.Analysis.AddWhy("Dev environment didn't have realistic data volume.")
	report.Analysis.AddWhy("We lack automated load testing in CI/CD.")

	fmt.Println(report.GenerateMarkdown())
}
```

## Interview Questions
**Q: What is the most important output of a post-mortem?**
**A:** Actionable engineering tasks. A post-mortem without Jira tickets is just a diary entry. We must change the system to prevent the same class of error from happening again.

**Q: How do you handle a post-mortem where an engineer clearly made a careless mistake?**
**A:** I focus on the system. Why did the system allow a "careless mistake" to take down production? Where were the guardrails, the linters, the code reviews, or the canary deployments? The failure is always in the safeguards.

**Q: Who should attend a post-mortem?**
**A:** The people involved in the incident, the service owners, and stakeholders. It should be open to the engineering org for learning, but the core analysis group should be small enough to be effective.
