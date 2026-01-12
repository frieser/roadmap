# Impact Assessment

## Summary
Impact Assessment is the "Look before you leap" phase of change management. It involves analyzing who and what will be affected by a proposed change. This prevents unintended consequences (e.g., "We fixed the billing code but broke the sales commission report").

## Detailed Explanation

### 1. Scope of Impact
*   **People**: Who has to work differently? (Devs? Sales? Support?).
*   **Process**: What workflows break? (Does the release script still work?).
*   **Technology**: What dependencies break? (API clients, integrations).

### 2. The "Blast Radius"
*   **Direct**: The team making the change.
*   **Indirect**: The teams consuming the API.
*   **Remote**: The Finance team consuming the data warehouse dump. (Often forgotten).

### 3. Severity Matrix
*   **Low Impact**: Minimal training needed (e.g., UI color change).
*   **Medium Impact**: Workflow change (e.g., new Jira field).
*   **High Impact**: Role/Job change (e.g., Reorg).

## Go Code Example: Dependency Graph Analyzer
This tool checks a graph of systems to find all downstream dependencies affected by a change.

```go
package main

import (
	"fmt"
)

type System struct {
	Name      string
	Consumers []string
}

func AnalyzeBlastRadius(changeTarget string, registry map[string]System) {
	fmt.Printf("--- Impact Analysis for: %s ---\n", changeTarget)
	
	affected := make(map[string]bool)
	queue := []string{changeTarget}
	
	// BFS to find all downstream consumers
	for len(queue) > 0 {
		current := queue[0]
		queue = queue[1:]
		
		sys := registry[current]
		for _, consumer := range sys.Consumers {
			if !affected[consumer] {
				fmt.Printf("-> Impacts: %s\n", consumer)
				affected[consumer] = true
				queue = append(queue, consumer)
			}
		}
	}
	
	if len(affected) == 0 {
		fmt.Println("✅ Low Impact: Isolated change.")
	} else {
		fmt.Printf("\n⚠️  Total Affected Systems: %d\n", len(affected))
	}
}

func main() {
	registry := map[string]System{
		"AuthService": {Name: "AuthService", Consumers: []string{"Billing", "Frontend", "MobileApp"}},
		"Billing":     {Name: "Billing", Consumers: []string{"FinanceReport", "EmailService"}},
		"Frontend":    {Name: "Frontend", Consumers: []string{}},
	}

	AnalyzeBlastRadius("AuthService", registry)
}
```

## Interview Questions

### Q: "How do you uncover 'hidden' dependencies?"
**A:**
*   **Scream Test**: (Risky).
*   **Interviews**: Talk to the longest-tenured engineers ("The Historians").
*   **Logs**: Check access logs to see who is actually calling the API.

### Q: "You assessed the impact as Low, but it turned out High. What happened?"
**A:**
*   **Assumption Failure**: I assumed people used the tool as documented. They hacked it to do something else.
*   **Correction**: Post-mortem the assessment process. Add "User Interviews" to the checklist next time.

### Q: "Why assess impact on non-engineering teams?"
**A:**
*   Because Engineering supports the business. If we change the "Order Status" field, we might break the Support team's ability to refund customers. That is a business outage.
