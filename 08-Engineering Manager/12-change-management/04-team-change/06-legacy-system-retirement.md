# Legacy System Retirement

## Summary
Legacy System Retirement is the end-of-life (EOL) phase of a technology. It is a change management challenge because users (and sometimes engineers) are emotionally or technically attached to the old system. The goal is to shut it down without business interruption.

## Detailed Explanation

### 1. The Sunset Policy
*   Announce EOL date far in advance (e.g., 6-12 months).
*   **Brownouts**: Intentionally degrade the service (e.g., add 5s latency or turn it off for 1 hour) to see who screams. This identifies undocumented dependencies.

### 2. Read-Only Mode
*   Before deleting data, make the system Read-Only.
*   This prevents new data creation while allowing archival access.

### 3. Celebration
*   Retiring a system is a win (deleting code is better than writing code).
*   Host a "Funeral" party for the old server. Print out the code and burn it (safely). Closure matters.

## Go Code Example: Feature Flag Deprecation
This code demonstrates using a feature flag to control the rollout of a deprecation warning.

```go
package main

import (
	"fmt"
	"time"
)

type User struct {
	ID string
}

func CallLegacyAPI(u User, eolDate time.Time) {
	now := time.Now()
	
	if now.After(eolDate) {
		fmt.Println("HTTP 410 GONE: This API is shut down.")
		return
	}

	// Brownout Phase: 1 week before
	if eolDate.Sub(now) < 7*24*time.Hour {
		fmt.Println("HTTP 200 (WARNING): API deprecating in < 7 days!")
		// Simulate random failure/latency
		time.Sleep(2 * time.Second) 
	} else {
		fmt.Println("HTTP 200 OK")
	}
}

func main() {
	eol := time.Now().Add(3 * 24 * time.Hour) // EOL in 3 days
	user := User{ID: "123"}

	fmt.Println("--- Client Calling Legacy API ---")
	CallLegacyAPI(user, eol)
}
```

## Interview Questions

### Q: "How do you convince a customer to move off a legacy product?"
**A:**
*   **Carrot**: "The new system is faster and has feature X."
*   **Stick**: "The old system will no longer receive security patches."
*   **Bridge**: "We have built a migration tool to copy your data automatically."

### Q: "Why is deleting code harder than writing it?"
**A:**
*   **Chesterton's Fence**: You don't know *why* the code was put there. It might handle a weird edge case.
*   **Fear**: "If I touch it, I break it."
*   **Solution**: Good tests give confidence to delete.

### Q: "What is a 'Scream Test'?"
**A:**
*   Turning off a server/service for a short time to see who complains.
*   *Caveat*: Only do this during business hours when you are ready to rollback instantly!
