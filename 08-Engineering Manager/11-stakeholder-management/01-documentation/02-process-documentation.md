## Summary
Process Documentation covers the "How We Work" aspect: Onboarding, Code Review Guidelines, Release Procedures, and Incident Response. It standardizes behavior, ensuring consistency and quality regardless of which engineer performs the task.

## Detailed Explanation
Process docs reduce cognitive load. You shouldn't have to "remember" how to release to production; you should follow a checklist.

### Vital Docs
1.  **Onboarding Guide**: "Day 1 to Day 30" plan.
2.  **SDLC**: How a ticket becomes code.
3.  **On-Call Runbooks**: "If Alert X fires, do Y."

## Go Code Example
Modeling a `ProcessEngine` that enforces a checklist workflow before an action can complete.

```go
package main

import "fmt"

type Step struct {
	Description string
	Completed   bool
}

type Checklist struct {
	Name  string
	Steps []Step
}

func (c *Checklist) MarkComplete(index int) {
	if index < len(c.Steps) {
		c.Steps[index].Completed = true
	}
}

func (c Checklist) IsReady() bool {
	for _, s := range c.Steps {
		if !s.Completed {
			return false
		}
	}
	return true
}

func main() {
	releaseChecklist := Checklist{
		Name: "Production Release",
		Steps: []Step{
			{Description: "Unit Tests Passed"},
			{Description: "Staging Verified"},
			{Description: "DB Backup Taken"},
		},
	}

	releaseChecklist.MarkComplete(0)
	releaseChecklist.MarkComplete(1)

	if releaseChecklist.IsReady() {
		fmt.Println("🚀 Deploying...")
	} else {
		fmt.Println("🛑 Halt! Checklist incomplete.")
	}
}
```

## Interview Questions
**Q: How do you handle a team that hates writing documentation?**
**A:** I lower the friction. I ask for bullet points instead of essays. I embed docs in the tools they use (e.g., PR templates, Slack workflows). I also lead by example—if I'm asked a question, I write the answer in a doc and link it.

**Q: What makes a "good" runbook?**
**A:** It is actionable. It doesn't explain the history of the database; it says "Step 1: Run this query. Step 2: If result > 100, scale up."
