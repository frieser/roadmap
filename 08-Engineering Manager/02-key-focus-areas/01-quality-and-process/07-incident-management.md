# Incident Management

## Summary
Incident Management is the structured process of identifying, analyzing, and correcting hazards to prevent a future re-occurrence. Effective incident management minimizes downtime (MTTR - Mean Time To Recovery) and transforms failures into learning opportunities through blameless post-mortems and rigorous root cause analysis.

## Detailed Explanation
In high-availability systems, failure is inevitable. Incident management is about how you handle that failure.

### The Incident Lifecycle
1.  **Detection**: Monitoring systems trigger an alert, or a user reports an issue.
2.  **Triage**: determining the severity (SEV1, SEV2, SEV3) and impact.
3.  **Response**:
    *   **Incident Commander (IC)**: The leader who coordinates the response.
    *   **Communications Lead**: Updates internal stakeholders and customers.
    *   **Tech Lead/Subject Matter Experts**: The engineers debugging the system.
4.  **Mitigation**: Stopping the bleeding (e.g., rolling back a deploy, enabling a circuit breaker). This comes *before* finding the root cause.
5.  **Resolution**: Full restoration of service.
6.  **Post-Mortem**: A "blameless" retrospective to understand *why* it happened and create action items to prevent recurrence.

### Key Metrics
- **MTTD (Mean Time To Detect)**: How long until we know there's a problem?
- **MTTA (Mean Time To Acknowledge)**: How long until a human starts looking at it?
- **MTTR (Mean Time To Recover)**: How long until service is restored?

### Best Practices
- **War Rooms**: Dedicated Slack channels or Zoom calls for active incidents.
- **Blameless Culture**: Focus on process/system failure, not human error. "You can't fire your way to reliability."
- **Game Days**: Simulating failures (Chaos Engineering) to practice the response.

## Go Code Example
This example models an Incident Management System's core logic: calculating severity based on impact and tracking the incident timeline (MTTD/MTTR).

```go
package main

import (
	"fmt"
	"time"
)

// Severity levels
const (
	Sev1 = "SEV1 - Critical" // System down, data loss
	Sev2 = "SEV2 - High"     // Major feature broken, workaround available
	Sev3 = "SEV3 - Medium"   // Minor bug, customer annoyed
)

// Incident represents a production issue
type Incident struct {
	ID          string
	Description string
	StartTime   time.Time
	DetectTime  time.Time
	ResolveTime time.Time
	Affected    int // Number of users affected
	IsDataLoss  bool
}

// IncidentCalculator contains logic for metrics and triage
type IncidentCalculator struct{}

// DetermineSeverity automatically suggests a severity based on heuristics
func (c *IncidentCalculator) DetermineSeverity(inc Incident) string {
	if inc.IsDataLoss {
		return Sev1
	}
	if inc.Affected > 10000 {
		return Sev1
	}
	if inc.Affected > 1000 {
		return Sev2
	}
	return Sev3
}

// CalculateMetrics returns the key KPI durations
func (c *IncidentCalculator) CalculateMetrics(inc Incident) (mttd time.Duration, mttr time.Duration) {
	mttd = inc.DetectTime.Sub(inc.StartTime)
	mttr = inc.ResolveTime.Sub(inc.StartTime)
	return mttd, mttr
}

func main() {
	calc := IncidentCalculator{}

	// Scenario: A bad deploy caused a DB lock
	start := time.Date(2023, 10, 1, 14, 0, 0, 0, time.UTC)
	detect := start.Add(5 * time.Minute)   // Took 5 mins to alert
	resolve := start.Add(45 * time.Minute) // Fixed 40 mins later

	incident := Incident{
		ID:          "INC-2023-10-01",
		Description: "Database deadlock on payment table",
		StartTime:   start,
		DetectTime:  detect,
		ResolveTime: resolve,
		Affected:    15000,
		IsDataLoss:  false,
	}

	// 1. Triage
	severity := calc.DetermineSeverity(incident)
	fmt.Printf("Incident: %s\n", incident.Description)
	fmt.Printf("Auto-Assessed Severity: %s\n", severity)

	// 2. Metrics
	mttd, mttr := calc.CalculateMetrics(incident)
	fmt.Printf("MTTD (Time to Detect): %s\n", mttd)
	fmt.Printf("MTTR (Time to Recover): %s\n", mttr)

	// 3. SLA Check
	slaLimit := 1 * time.Hour
	if mttr > slaLimit {
		fmt.Println("❌ SLA Breached")
	} else {
		fmt.Println("✅ Within SLA")
	}
}
```

## Interview Questions
1.  **Describe the role of an Incident Commander. Why is it important to have one designated person?**
    *   *Focus*: Coordination, avoiding the "bystander effect," clear communication channels, decision-making authority.
2.  **What does a "blameless post-mortem" mean to you? Why is it crucial?**
    *   *Focus*: Psychological safety, root cause analysis (5 Whys), preventing recurrence vs. punishing individuals.
3.  **How do you handle communication during a SEV1 outage?**
    *   *Focus*: Internal vs. external comms, frequency of updates, managing stakeholder expectations.
4.  **How do you reduce MTTR (Mean Time To Recovery)?**
    *   *Focus*: Better monitoring/alerting, automated rollbacks, runbooks, chaos engineering/practice.
