# Migration Planning

## Summary
Migration Planning is the strategic process of moving software, data, or infrastructure from one environment to another (e.g., On-Prem to Cloud, Monolith to Microservices). It is high-risk, high-reward. EMs must balance "keeping the lights on" with the "surgery" of migration, ensuring zero downtime and data integrity.

## Detailed Explanation

### 1. The Migration Strategy (The 6 Rs)
AWS defines 6 strategies:
*   **Rehost ("Lift and Shift")**: Move VM to VM without changes. Fast, but misses cloud benefits.
*   **Replatform**: Move to managed services (e.g., self-hosted MySQL to RDS).
*   **Refactor**: Rewrite code to be cloud-native (Microservices). Highest value, highest cost.
*   **Repurchase**: Move to SaaS (Drop custom CRM for Salesforce).
*   **Retire**: Turn off unused apps.
*   **Retain**: Do nothing (for now).

### 2. The Strangler Fig Pattern (Martin Fowler)
*   Never rewrite a big system from scratch (Big Bang).
*   Create a new system around the edges of the old one.
*   Gradually intercept calls and route them to the new system.
*   Eventually, the old system is strangled and can be decommissioned.

### 3. Data Migration Challenges
*   **Dual Write**: Write to Old and New DB simultaneously.
*   **Backfill**: Copy historical data to New DB.
*   **Validation**: Compare Old vs New reads to ensure accuracy (Shadow Mode).
*   **Cutover**: Switch reads to New DB.

## Go Code Example: Shadow Traffic Router
This example demonstrates "Shadow Mode" (Dark Launching). It sends traffic to the legacy system (Primary) but also asynchronously sends it to the new system to test performance/correctness without affecting the user.

```go
package main

import (
	"fmt"
	"time"
)

type Response struct {
	Body       string
	StatusCode int
}

// LegacySystem is the source of truth
func LegacySystem(req string) Response {
	time.Sleep(50 * time.Millisecond)
	return Response{"Legacy Result", 200}
}

// NewSystem is being tested
func NewSystem(req string) Response {
	time.Sleep(20 * time.Millisecond) // It's faster!
	return Response{"New Result", 200}
}

func Router(req string, shadowMode bool) Response {
	// 1. Always call Legacy (Critical Path)
	legacyRes := LegacySystem(req)

	// 2. If Shadow Mode, call New System asynchronously
	if shadowMode {
		go func() {
			newRes := NewSystem(req)
			// Compare results (Log mismatch, don't fail user)
			if newRes.Body != legacyRes.Body {
				fmt.Printf("⚠️ MISMATCH: Legacy='%s', New='%s'\n", legacyRes.Body, newRes.Body)
			} else {
				fmt.Println("✅ MATCH")
			}
		}()
	}

	return legacyRes
}

func main() {
	fmt.Println("--- Request 1 (Match) ---")
	Router("user_123", true)
	time.Sleep(100 * time.Millisecond) // Wait for goroutine

	// Simulate mismatch in logic
	fmt.Println("\n--- Request 2 (Mismatch) ---")
	// Mocking mismatch logic for demo...
	Router("user_999", true) 
	time.Sleep(100 * time.Millisecond)
}
```

## Interview Questions

### Q: "How do you handle a zero-downtime database migration?"
**A:**
*   **Expand-Contract Pattern**:
    1.  **Expand**: Add new column/table. Code writes to both (Dual Write).
    2.  **Migrate**: Backfill old data to new column.
    3.  **Contract**: Remove code reading old column. Remove old column.

### Q: "What is the biggest risk in a migration?"
**A:**
*   **Data Corruption**: Losing or corrupting customer data is irreversible.
*   **Mitigation**: Heavy testing in staging, Shadow Mode in prod, and always having a **Rollback Plan**.

### Q: "Why do you prefer Strangler Fig over Big Bang?"
**A:**
*   **Risk**: Big Bang requires a "switch flip" where everything must work instantly. If it fails, you have no service.
*   **Value**: Strangler Fig delivers value incrementally (module by module). Big Bang delivers zero value until the very end (which might be years later).
