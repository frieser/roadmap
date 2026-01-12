## Summary
Release Management is the process of managing, planning, scheduling, and controlling a software build through different stages and environments; including testing and deploying software releases. It ensures that delivery of new features does not compromise the integrity of the live environment.

## Detailed Explanation
Modern release management has shifted from infrequent, high-risk "Big Bang" releases to frequent, low-risk deployments (CI/CD).

### Strategies
*   **Blue-Green Deployment**: Two identical environments. Traffic is switched from Blue (old) to Green (new). Instant rollback possible.
*   **Canary Release**: Rolling out the update to a small subset of users (e.g., 5%) first to test stability before full rollout.
*   **Feature Flags**: Deploying code to production but keeping it hidden behind a configuration flag.

### Versioning
*   **Semantic Versioning (SemVer)**: MAJOR.MINOR.PATCH (e.g., 2.1.4).
    *   Major: Incompatible API changes.
    *   Minor: Backwards-compatible functionality.
    *   Patch: Backwards-compatible bug fixes.

## Go Code Example
A simple Release Manager struct that handles semantic versioning and simulates a Canary deployment rollout.

```go
package main

import (
	"fmt"
	"time"
)

type Version struct {
	Major, Minor, Patch int
}

func (v Version) String() string {
	return fmt.Sprintf("v%d.%d.%d", v.Major, v.Minor, v.Patch)
}

type Release struct {
	Tag      Version
	Features []string
	IsStable bool
}

type Deployer struct {
	CurrentTrafficPercent int
}

func (d *Deployer) RolloutCanary(r Release) {
	fmt.Printf("Starting Canary Rollout for %s...\n", r.Tag)
	
	steps := []int{5, 10, 25, 50, 100}
	
	for _, percent := range steps {
		d.CurrentTrafficPercent = percent
		fmt.Printf("Traffic shifted to: %d%%\n", d.CurrentTrafficPercent)
		
		// Simulate monitoring check
		if !d.HealthCheck() {
			fmt.Println("ALERT: Error spike detected! Rolling back immediately.")
			d.Rollback()
			return
		}
		time.Sleep(100 * time.Millisecond) // Simulating bake time
	}
	
	fmt.Println("Rollout Complete. New version is 100% live.")
}

func (d *Deployer) HealthCheck() bool {
	// Simulate a failure at 50% traffic
	if d.CurrentTrafficPercent == 50 {
		return false
	}
	return true
}

func (d *Deployer) Rollback() {
	d.CurrentTrafficPercent = 0
	fmt.Println("Rollback successful. Traffic is 0% on new version.")
}

func main() {
	rel := Release{
		Tag: Version{1, 2, 0},
		Features: []string{"Dark Mode", "Payment API"},
	}
	
	deployer := &Deployer{}
	deployer.RolloutCanary(rel)
}
```

## Interview Questions
**Q: What is the difference between Deployment and Release?**
**A:** Deployment is the technical act of moving code to an environment (e.g., moving binaries to a server). Release is the business activity of making features available to users. Using Feature Flags, you can deploy code on Tuesday but "release" the feature on Friday.

**Q: How do you decide between Blue-Green and Canary deployments?**
**A:** Blue-Green is better for atomic switchovers and instant full rollback but requires double the infrastructure. Canary is better for testing new code in production with minimal risk (blast radius) but takes longer to fully saturate.

**Q: Explain Semantic Versioning.**
**A:** It's a standard (Major.Minor.Patch) that communicates the risk of upgrading. Breaking changes increment Major, new features increment Minor, fixes increment Patch. It allows dependency managers to automatically pull safe updates.
