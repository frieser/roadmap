## Summary
Post-Incident Analysis (PIA) is the bridge between a crisis and future prevention. It is the formal document produced after the Blameless Post-Mortem meeting. It crystallizes the "What, Why, and How" into permanent institutional knowledge.

## Detailed Explanation
A PIA is not just a meeting; it's an artifact.

### Essential Sections
1.  **Summary**: Executive high-level view.
2.  **Impact**: Duration, users affected, revenue lost.
3.  **Timeline**: Minute-by-minute log (Detection, Diagnosis, Mitigation).
4.  **Root Cause**: Technical depth.
5.  **Lessons Learned**: What went well? What went poorly?
6.  **Action Items**: Jira links.

## Go Code Example
Modeling a `PostMortem` struct that calculates MTTR (Mean Time To Recovery) and MTTD (Mean Time To Detect).

```go
package main

import (
	"fmt"
	"time"
)

type IncidentStats struct {
	StartTime    time.Time
	DetectedTime time.Time
	ResolvedTime time.Time
}

func (i IncidentStats) CalculateMetrics() {
	mttd := i.DetectedTime.Sub(i.StartTime)
	mttr := i.ResolvedTime.Sub(i.StartTime)
	fixTime := i.ResolvedTime.Sub(i.DetectedTime)

	fmt.Printf("Time to Detect (MTTD): %v\n", mttd)
	fmt.Printf("Time to Resolve (MTTR): %v\n", mttr)
	fmt.Printf("Actual Fix Time: %v\n", fixTime)
}

func main() {
	stats := IncidentStats{
		StartTime:    time.Date(2023, 10, 1, 14, 0, 0, 0, time.UTC),
		DetectedTime: time.Date(2023, 10, 1, 14, 15, 0, 0, time.UTC), // 15 min lag
		ResolvedTime: time.Date(2023, 10, 1, 15, 0, 0, 0, time.UTC),  // 1 hour total
	}

	stats.CalculateMetrics()
}
```

## Interview Questions
**Q: How do you ensure Action Items from a PIA actually get done?**
**A:** I treat them as Technical Debt. They get prioritized in the backlog. For critical items (fixing the root cause), we might pause feature work in the next sprint. I also review open post-mortem actions in our weekly ops meeting.

**Q: What is the difference between a Root Cause and a Trigger?**
**A:** The Trigger is the immediate event (e.g., "High traffic"). The Root Cause is the flaw that made the system unable to handle the trigger (e.g., "Missing connection pooling"). We fix Root Causes, not Triggers.
