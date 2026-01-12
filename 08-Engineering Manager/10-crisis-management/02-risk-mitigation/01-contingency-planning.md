## Summary
Contingency Planning involves preparing for specific, foreseeable "bad days." Unlike general resilience, this is about having a "Plan B" (and "Plan C") for critical dependencies, key personnel loss, or vendor failures. It reduces panic by providing a pre-agreed playbook.

## Detailed Explanation
A contingency plan answers "What if X fails?" before X actually fails.

### Areas for Planning
1.  **Vendor Failure**: What if GitHub is down? What if AWS us-east-1 has an outage?
2.  **Key Person Risk**: What if the Tech Lead wins the lottery or gets sick?
3.  **Tool Failure**: What if our CI/CD pipeline breaks on release day?

### The Playbook
A good contingency plan is a checklist, not a novel. It should be accessible offline.

## Go Code Example
Modeling a `DependencyManager` with fallback strategies (Circuit Breaker pattern).

```go
package main

import (
	"errors"
	"fmt"
)

type Service func() (string, error)

type CircuitBreaker struct {
	Primary   Service
	Secondary Service
	Fallback  Service
}

func (cb CircuitBreaker) Execute() (string, error) {
	// Try Primary
	res, err := cb.Primary()
	if err == nil {
		return res, nil
	}
	fmt.Println("Primary failed, switching to Secondary...")

	// Try Secondary (Contingency Plan A)
	res, err = cb.Secondary()
	if err == nil {
		return res, nil
	}
	fmt.Println("Secondary failed, switching to Fallback...")

	// Try Fallback (Contingency Plan B - e.g., cached data)
	return cb.Fallback()
}

func main() {
	plan := CircuitBreaker{
		Primary: func() (string, error) { return "", errors.New("AWS Down") },
		Secondary: func() (string, error) { return "", errors.New("Azure Down") },
		Fallback: func() (string, error) { return "Serving Static Site", nil },
	}

	result, _ := plan.Execute()
	fmt.Println("Final Result:", result)
}
```

## Interview Questions
**Q: How do you identify which risks require a contingency plan?**
**A:** I use a Risk Matrix (Likelihood vs. Impact). High Impact/High Likelihood risks get immediate mitigation. High Impact/Low Likelihood risks get contingency plans.

**Q: Give an example of a contingency plan you've created.**
**A:** We relied heavily on a 3rd party SMS provider. I mandated we integrate a secondary provider and wrote a "switchover" script. When the primary went down on Black Friday, we flipped the switch in 5 minutes.
