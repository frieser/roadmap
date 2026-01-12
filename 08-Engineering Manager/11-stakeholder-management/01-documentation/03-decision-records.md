## Summary
Architecture Decision Records (ADRs) are short text files that capture significant architectural decisions, along with their context and consequences. They answer the question "Why did we do it this way?" for future engineers (and your future self).

## Detailed Explanation
An ADR typically contains:
*   **Status**: Proposed, Accepted, Deprecated.
*   **Context**: The problem we were solving.
*   **Decision**: What we chose.
*   **Consequences**: The pros and cons of that choice.

### Why use them?
They prevent "Chesterton's Fence" scenarios where people are afraid to change old code because they don't know why it exists.

## Go Code Example
Modeling an `ADRRegistry` that parses markdown files to find the decision status of a component.

```go
package main

import (
	"fmt"
	"strings"
)

type ADRStatus string

const (
	StatusAccepted   ADRStatus = "ACCEPTED"
	StatusDeprecated ADRStatus = "DEPRECATED"
)

type ADR struct {
	ID       int
	Title    string
	Status   ADRStatus
	Decision string
}

func (a ADR) Summary() string {
	return fmt.Sprintf("[%d] %s: %s", a.ID, a.Title, a.Status)
}

func main() {
	// Simulating parsing a folder of markdown files
	adrs := []ADR{
		{ID: 1, Title: "Use Postgres for Users", Status: StatusAccepted, Decision: "Use Relational DB"},
		{ID: 2, Title: "Use MongoDB for Logs", Status: StatusDeprecated, Decision: "Use NoSQL"},
	}

	fmt.Println("Active Architectural Decisions:")
	for _, adr := range adrs {
		if adr.Status == StatusAccepted {
			fmt.Println(adr.Summary())
		}
	}
}
```

## Interview Questions
**Q: When should you write an ADR?**
**A:** Whenever a decision has a significant impact (e.g., introducing a new language, database, or API standard) or is difficult to reverse. Routine bug fixes don't need ADRs.

**Q: How do you encourage the team to write ADRs?**
**A:** I integrate it into the RFC process. You can't merge a major architectural change without an ADR file in the `docs/adr` folder. It becomes part of the Definition of Done.
