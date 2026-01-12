## Summary
Disaster Recovery (DR) is the subset of contingency planning focused specifically on restoring IT infrastructure and data after a catastrophic event (data center loss, massive cyberattack, natural disaster). Key metrics are RTO (Recovery Time Objective) and RPO (Recovery Point Objective).

## Detailed Explanation
DR is about survival. It requires redundancy and rigorous testing.

### Key Metrics
*   **RTO (Recovery Time Objective)**: How long can you afford to be down? (e.g., 4 hours).
*   **RPO (Recovery Point Objective)**: How much data can you afford to lose? (e.g., 15 minutes).

### Strategies
*   **Active-Passive**: Standby environment ready to boot up.
*   **Active-Active**: Traffic load-balanced across regions; if one fails, the other takes over.
*   **Pilot Light**: Minimal critical core running, scales up during disaster.

## Go Code Example
Modeling a `DataReplicator` that enforces RPO compliance by checking replication lag.

```go
package main

import (
	"fmt"
	"time"
)

type Database struct {
	Region      string
	LastCommit  time.Time
}

type DRMonitor struct {
	MaxRPO time.Duration
}

func (dr DRMonitor) CheckRPO(primary, replica Database) bool {
	lag := primary.LastCommit.Sub(replica.LastCommit)
	fmt.Printf("Replication Lag: %v (Max Allowed: %v)\n", lag, dr.MaxRPO)
	
	if lag > dr.MaxRPO {
		return false // RPO VIOLATION
	}
	return true
}

func main() {
	now := time.Now()
	primary := Database{Region: "us-east-1", LastCommit: now}
	// Replica is 20 minutes behind
	replica := Database{Region: "us-west-2", LastCommit: now.Add(-20 * time.Minute)}

	monitor := DRMonitor{MaxRPO: 15 * time.Minute}

	if !monitor.CheckRPO(primary, replica) {
		fmt.Println("🚨 CRITICAL: RPO Violation! Trigger backup snapshot restore.")
	} else {
		fmt.Println("✅ System healthy.")
	}
}
```

## Interview Questions
**Q: What is the difference between RTO and RPO?**
**A:** RTO is about *time* (how long until we are back up?), RPO is about *data* (how much work did we lose?). You might be back up in 1 minute (great RTO) but have lost 1 day of data (terrible RPO).

**Q: How often should you test your DR plan?**
**A:** At least annually, but ideally quarterly. "Game Days" where we simulate a region failure in staging (or production if mature) are essential to verify the runbooks actually work.
