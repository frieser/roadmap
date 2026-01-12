# Process Changes

## Summary
Process Changes (e.g., changing code review policy, altering sprint length) modify *how* work gets done. The goal is to reduce friction, but any change adds friction initially. EMs must treat process as a product: iterate, measure, and pivot.

## Detailed Explanation

### 1. Descriptve vs. Prescriptive
*   **Prescriptive**: "You must do X." (Top-down).
*   **Descriptive**: "We noticed X works well." (Bottom-up).
*   *Advice*: Start descriptive. "Team A tried Kanban and shipped faster. Should we try it?"

### 2. Retrospectives as Engines of Change
*   Don't change process because *you* read a book.
*   Change process because *the team* complained about a problem in the Retro.
*   "You said meetings are too long. Let's try async standups."

### 3. Measuring Impact
*   If you change the process, measure the metric it was supposed to fix.
*   "We switched to Trunk Based Development to improve cycle time. Did cycle time go down?"

## Go Code Example: Process Policy Validator
Checks if a PR meets the new process requirements (e.g., Title format, Test coverage).

```go
package main

import (
	"fmt"
	"strings"
)

type PullRequest struct {
	Title        string
	HasTests     bool
	Approvals    int
}

func CheckProcess(pr PullRequest) {
	fmt.Printf("Checking PR: %s\n", pr.Title)
	
	// Rule 1: Conventional Commits
	if !strings.HasPrefix(pr.Title, "feat:") && !strings.HasPrefix(pr.Title, "fix:") {
		fmt.Println("❌ Violation: Title must start with 'feat:' or 'fix:'")
		return
	}

	// Rule 2: Tests Required
	if !pr.HasTests {
		fmt.Println("❌ Violation: No tests detected.")
		return
	}

	// Rule 3: 2 Approvals
	if pr.Approvals < 2 {
		fmt.Println("❌ Violation: Needs 2 approvals.")
		return
	}

	fmt.Println("✅ Process Adhered.")
}

func main() {
	badPR := PullRequest{"Added logic", false, 1}
	goodPR := PullRequest{"feat: Added login logic", true, 2}

	CheckProcess(badPR)
	fmt.Println("---")
	CheckProcess(goodPR)
}
```

## Interview Questions

### Q: "The team ignores the new process. What do you do?"
**A:**
*   **Is the process bad?**: If smart people ignore a rule, the rule is usually dumb/toilsome.
*   **Automate**: Use linters/CI to enforce it. Robots don't get ignored.
*   **Re-evaluate**: Ask "Is this rule still serving us?"

### Q: "How do you introduce a 'Heavy' process (like Security Reviews) to a 'Agile' team?"
**A:**
*   **Explain the 'Why'**: "We are SOC2 compliant now. We have to do this."
*   **Make it easy**: Templates, Checklists.
*   **Gate**: "You can't deploy without it." (Hard gate).

### Q: "What is 'Process Debt'?"
**A:**
*   Rules that exist for a problem that no longer exists.
*   *Example*: "We require VP approval for deploying" (from when we broke prod 2 years ago).
*   *Action*: Prune process regularly.
