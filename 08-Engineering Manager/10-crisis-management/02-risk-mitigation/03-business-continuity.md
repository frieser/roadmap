## Summary
Business Continuity Planning (BCP) is broader than Disaster Recovery. While DR focuses on IT systems, BCP focuses on the *business operations*. It covers people, communication, offices, and legal requirements. It ensures the company continues to make money and serve customers even if the headquarters burns down.

## Detailed Explanation
BCP asks: "If we have no laptops, no internet, and no office, how do we process payroll?"

### Elements of BCP
1.  **People Safety**: Emergency contact lists, evacuation plans.
2.  **Remote Work Readiness**: Can everyone work from home indefinitely?
3.  **Succession Planning**: Who is in charge if the CEO is unavailable?
4.  **Critical Function Identification**: What *must* run vs. what can wait?

## Go Code Example
Modeling a `BusinessProcess` priority queue to determine what services to restore first in a constrained environment.

```go
package main

import (
	"fmt"
	"sort"
)

type Criticality int

const (
	MissionCritical Criticality = iota // 0
	BusinessCritical                   // 1
	InternalSupport                    // 2
	NiceToHave                         // 3
)

type Process struct {
	Name        string
	Level       Criticality
	ResourceCost int // Resources needed to run
}

type ContinuityManager struct {
	AvailableResources int
}

func (cm ContinuityManager) Prioritize(processes []Process) []string {
	// Sort by criticality (lower is more critical)
	sort.Slice(processes, func(i, j int) bool {
		return processes[i].Level < processes[j].Level
	})

	var running []string
	currentLoad := 0

	for _, p := range processes {
		if currentLoad+p.ResourceCost <= cm.AvailableResources {
			running = append(running, p.Name)
			currentLoad += p.ResourceCost
		} else {
			fmt.Printf("Skipping %s (Not enough resources)\n", p.Name)
		}
	}
	return running
}

func main() {
	procs := []Process{
		{"Payroll", BusinessCritical, 20},
		{"Customer Checkout", MissionCritical, 50},
		{"Employee Blog", NiceToHave, 5},
		{"Inventory System", BusinessCritical, 30},
	}

	// Disaster scenario: Only 60% resources available
	mgr := ContinuityManager{AvailableResources: 60}
	
	active := mgr.Prioritize(procs)
	fmt.Printf("Active Processes: %v\n", active)
}
```

## Interview Questions
**Q: How does BCP differ from DR?**
**A:** DR is technical (servers, data); BCP is operational (people, payroll, legal). BCP uses DR as a tool. You can have a working database (DR success) but no one to answer support calls (BCP failure).

**Q: How do you handle communication during a BCP event?**
**A:** We use an out-of-band communication channel (e.g., a WhatsApp group or external status page) that doesn't rely on our internal corporate network, which might be down.
