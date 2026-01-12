## Summary
Team Support during a crisis is about psychological safety and workload management. A crisis puts high pressure on engineers, leading to burnout if not managed. The manager's role is to shield the team from external chaos, ensure they take breaks, and provide emotional support, reinforcing that they are working the problem, not *being* the problem.

## Detailed Explanation
In a crisis (like a major outage or security breach), cognitive load spikes. A manager must act as a "Chief Wellness Officer" for the duration.

### Key Actions
1.  **Shift Rotations**: Force breaks. No one makes good decisions after 12 hours of debugging.
2.  **Food and Logistics**: If physical, order food. If remote, tell people to step away for meals.
3.  **Shielding**: Intercept executive questions so the team can focus on the fix.
4.  **Aftercare**: Once resolved, enforce time off.

## Go Code Example
Modeling a `ShiftManager` that tracks engineer fatigue and enforces mandatory cooldowns.

```go
package main

import (
	"fmt"
	"time"
)

type Engineer struct {
	Name           string
	ActiveSince    time.Time
	OnShift        bool
	FatigueLevel   int // 0-100
}

type ShiftManager struct {
	MaxShiftDuration time.Duration
	Team             []*Engineer
}

func (sm *ShiftManager) CheckHealth() {
	for _, eng := range sm.Team {
		if !eng.OnShift {
			continue
		}
		
		shiftDuration := time.Since(eng.ActiveSince)
		if shiftDuration > sm.MaxShiftDuration {
			fmt.Printf("⚠️ ALERT: %s has been on shift for %v. MANDATORY BREAK REQUIRED.\n", eng.Name, shiftDuration)
			eng.FatigueLevel = 100
		} else {
			fmt.Printf("✅ %s is okay (Shift: %v)\n", eng.Name, shiftDuration)
		}
	}
}

func main() {
	manager := ShiftManager{
		MaxShiftDuration: 4 * time.Hour, // Strict 4-hour shifts during crisis
		Team: []*Engineer{
			{Name: "Alice", ActiveSince: time.Now().Add(-5 * time.Hour), OnShift: true},
			{Name: "Bob", ActiveSince: time.Now().Add(-2 * time.Hour), OnShift: true},
		},
	}

	manager.CheckHealth()
}
```

## Interview Questions
**Q: How do you prevent burnout during a prolonged incident?**
**A:** I establish a rigorous roster immediately. I designate a "secondary" on-call rotation to relieve the primary responders every 4-6 hours, and I explicitly order people to sign off, disabling their notifications.

**Q: An engineer caused an outage and is distraught. What do you do?**
**A:** I pull them aside immediately to reassure them: "We fix the system, we don't blame the person." I keep them involved in the fix (if they are able) to restore their confidence, but give them a less critical task if they are too shaken.
