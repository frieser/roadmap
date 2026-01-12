# Legacy System Retirement

## Summary
Legacy System Retirement (or Decommissioning) is the strategic process of identifying, migrating away from, and shutting down obsolete software systems. It is crucial for reducing technical debt, security risks, and operational costs. The "Strangler Fig" pattern is the most common and effective strategy for this.

## Detailed Explanation
Old code never dies; it just becomes "legacy." Legacy systems are often profitable but brittle, hard to change, and run on outdated tech stacks.

### Why Retire Systems?
- **Cost**: Hosting fees, licensing, and expensive maintenance hours.
- **Risk**: Security vulnerabilities in unpatched dependencies.
- **Velocity**: Legacy systems often lack CI/CD and tests, slowing down the team.

### The "Strangler Fig" Pattern
Instead of a "Big Bang" rewrite (which usually fails), use the Strangler Fig pattern:
1.  Identify a specific slice of functionality in the legacy system.
2.  Build a new microservice/module for just that slice.
3.  Route *new* traffic to the new service; fallback to old if needed.
4.  Repeat until the legacy system is doing nothing.
5.  Turn off the legacy system.

### The Screaming Test
If you aren't sure who uses a service, turn it off (or block access) for a short time and see who "screams." (Use with extreme caution).

## Go Code Example
This example demonstrates the **Strangler Fig Pattern** at the routing layer. It creates a proxy that gradually migrates traffic from a `LegacyUrl` to a `ModernUrl` based on a feature flag or migration percentage.

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

// StranglerFacade manages routing between systems
type StranglerFacade struct {
	LegacySystemURL string
	ModernSystemURL string
	MigrationPercent int // 0 to 100
}

func (s *StranglerFacade) HandleRequest(reqID int) {
	// Randomly decide routing based on migration percentage
	if s.shouldUseModern() {
		fmt.Printf("[Req %d] -> ✨ Routing to MODERN system (%s)\n", reqID, s.ModernSystemURL)
	} else {
		fmt.Printf("[Req %d] -> 👴 Routing to LEGACY system (%s)\n", reqID, s.LegacySystemURL)
	}
}

func (s *StranglerFacade) shouldUseModern() bool {
	// Simple percentage check
	r := rand.Intn(100)
	return r < s.MigrationPercent
}

func main() {
	rand.Seed(time.Now().UnixNano())

	router := StranglerFacade{
		LegacySystemURL: "http://monolith.internal/api/v1/users",
		ModernSystemURL: "http://users-service.internal/v2/users",
		MigrationPercent: 0,
	}

	fmt.Println("--- Phase 1: 0% Migration (Safety Check) ---")
	for i := 0; i < 5; i++ {
		router.HandleRequest(i)
	}

	fmt.Println("\n--- Phase 2: 30% Canary Rollout ---")
	router.MigrationPercent = 30
	for i := 0; i < 10; i++ {
		router.HandleRequest(i)
	}

	fmt.Println("\n--- Phase 3: 100% Cutover ---")
	router.MigrationPercent = 100
	for i := 0; i < 5; i++ {
		router.HandleRequest(i)
	}
	
	fmt.Println("\n[Status] Legacy System can now be decommissioned.")
}
```

## Interview Questions
1.  **Why is a "Big Bang" rewrite usually a bad idea?**
    *   *Focus*: High risk, long time to value, feature gap (legacy has hidden features), business stagnation during the rewrite.
2.  **Explain the Strangler Fig pattern.**
    *   *Focus*: Gradual replacement, event interception, risk reduction.
3.  **How do you handle data synchronization during a migration?**
    *   *Focus*: Dual writing (write to both, read from one), Change Data Capture (CDC), eventual consistency challenges.
4.  **How do you convince stakeholders to invest time in retiring a system that "works fine"?**
    *   *Focus*: Highlight TCO (maintenance cost), risk (security/bus factor), and opportunity cost (team speed).
