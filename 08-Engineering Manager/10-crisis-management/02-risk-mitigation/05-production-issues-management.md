## Summary
Production Issues Management is the daily hygiene of handling bugs and minor outages that don't rise to the level of a crisis. It involves triage, prioritization, and establishing an on-call rotation that isn't miserable.

## Detailed Explanation
Not every bug is an incident. Effective management is about filtration.

### Best Practices
*   **Triage Process**: A designated "Triage Officer" reviews incoming tickets daily.
*   **Severity Levels**: Clear definitions (SEV1 = Drop everything, SEV2 = Fix today, SEV3 = Next sprint).
*   **Runbooks**: Step-by-step guides for common alerts.
*   **Alert Fatigue**: If an alert fires and no action is needed, delete the alert.

## Go Code Example
Modeling a `TriageQueue` that sorts issues by severity and SLA (Service Level Agreement).

```go
package main

import (
	"fmt"
	"sort"
	"time"
)

type Severity int

const (
	SEV1 Severity = 1 // Critical - 1 hour SLA
	SEV2 Severity = 2 // Major - 24 hour SLA
	SEV3 Severity = 3 // Minor - 1 week SLA
)

type Ticket struct {
	ID        string
	Sev       Severity
	Created   time.Time
}

// SLAHours returns the allowed resolution time
func (s Severity) SLAHours() float64 {
	switch s {
	case SEV1: return 1.0
	case SEV2: return 24.0
	default: return 168.0 // 1 week
	}
}

// TimeRemaining calculates hours left before SLA breach
func (t Ticket) TimeRemaining() float64 {
	deadline := t.Created.Add(time.Duration(t.Sev.SLAHours()) * time.Hour)
	return time.Until(deadline).Hours()
}

func main() {
	queue := []Ticket{
		{"BUG-101", SEV2, time.Now().Add(-20 * time.Hour)}, // 4 hours left
		{"BUG-102", SEV1, time.Now().Add(-10 * time.Minute)}, // 50 mins left
		{"BUG-103", SEV3, time.Now().Add(-48 * time.Hour)}, // Plenty of time
	}

	// Sort by "Time Remaining" (Urgency)
	sort.Slice(queue, func(i, j int) bool {
		return queue[i].TimeRemaining() < queue[j].TimeRemaining()
	})

	fmt.Println("Triage Priority:")
	for _, t := range queue {
		fmt.Printf("[%s] %s (%.1f hours remaining)\n", 
			t.ID, 
			map[Severity]string{1: "SEV1", 2: "SEV2", 3: "SEV3"}[t.Sev], 
			t.TimeRemaining())
	}
}
```

## Interview Questions
**Q: How do you handle an engineer who ignores on-call alerts?**
**A:** I investigate why. Is it alert fatigue (too many false positives)? Lack of confidence/training? Or negligence? Usually, it's the system (bad alerts). I work to clean up the alerts first, then address performance if the behavior continues.

**Q: What is your philosophy on "Zero Bugs"?**
**A:** It's an aspiration, not a reality. I prioritize "Zero Critical Bugs." Lower priority bugs are backlog items that we weigh against feature work. If we fix every cosmetic bug, we never ship features.
