## Summary
Service Recovery involves the technical steps to restore service health. This includes rollbacks, failovers, capacity increases, or feature toggling. The golden rule is: "Restore service first, debug later."

## Detailed Explanation
Recovery is not about fixing the *bug*; it's about stopping the *pain*.

### Strategies
1.  **Rollback**: The fastest fix for a bad deploy.
2.  **Scale Up**: Throwing money (servers) at the problem to buy time.
3.  **Degrade Gracefully**: Disable the heavy "Recommendations" widget so the "Checkout" button still works.
4.  **Restart**: The classic "turn it off and on again" often clears transient memory leaks.

## Go Code Example
Modeling a `FeatureFlag` system for graceful degradation.

```go
package main

import "fmt"

type ServiceState struct {
	Healthy bool
	Load    int // 0-100
}

type FeatureFlag struct {
	Name      string
	Enabled   bool
	RiskLevel int // Higher is riskier
}

func (s *ServiceState) Recover(flags map[string]*FeatureFlag) {
	fmt.Printf("Current Load: %d%%. Attempting recovery...\n", s.Load)
	
	// Strategy: Disable risky features to reduce load
	for _, flag := range flags {
		if flag.Enabled && flag.RiskLevel > 5 {
			fmt.Printf("Disabling feature: %s\n", flag.Name)
			flag.Enabled = false
			s.Load -= 20 // Simulated load reduction
		}
	}
	
	if s.Load < 80 {
		s.Healthy = true
		fmt.Println("✅ Service recovered to healthy levels.")
	} else {
		fmt.Println("❌ Service still overloaded.")
	}
}

func main() {
	state := &ServiceState{Healthy: false, Load: 95}
	flags := map[string]*FeatureFlag{
		"Core":           {Name: "Core API", Enabled: true, RiskLevel: 1},
		"Recommendations": {Name: "AI Recommendations", Enabled: true, RiskLevel: 9},
	}

	state.Recover(flags)
}
```

## Interview Questions
**Q: When should you rollback vs. roll forward (fix in place)?**
**A:** almost always Rollback. Rollback brings you to a known good state. Rolling forward involves writing new code under extreme pressure, which usually introduces new bugs.

**Q: What is a "Canary Release" and how does it aid recovery?**
**A:** It releases code to a small % of users first. If metrics tank, you only impact 1% of users and recovery is instantaneous (route traffic back to the main stable fleet).
