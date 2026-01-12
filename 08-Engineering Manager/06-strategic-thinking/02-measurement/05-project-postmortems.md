# Project Post-Mortems

## Summary
A Project Post-Mortem (or Retrospective) is a structured meeting held after a project launch or a major incident to analyze what went right, what went wrong, and how to improve. The core principle is **"Blamelessness"**—focusing on systemic failures rather than human error to prevent recurrence.

## Detailed Explanation

### 1. The Blameless Philosophy
*   **Assumption**: Everyone did the best they could with the information they had at the time.
*   **Goal**: Improve the *system* so that a well-intentioned human cannot make the same mistake again.
*   **Bad**: "Fire Bob because he clicked the wrong button."
*   **Good**: "Add a confirmation guard rail so nobody can click that button by accident."

### 2. Root Cause Analysis (The "5 Whys")
A technique to drill down to the fundamental cause.
*   *Problem*: The site crashed.
*   *Why?* The DB ran out of connections.
*   *Why?* The web server retried infinitely.
*   *Why?* We had no connection timeout configured.
*   *Why?* We used the default library settings.
*   *Root Cause*: Lack of standard configuration review/linting for libraries.

### 3. Key Outputs
A post-mortem is useless without **Action Items**.
*   **Fix**: Repair the immediate damage.
*   **Prevent**: Stop it from happening again.
*   **Detect**: Improve monitoring to catch it faster next time.

### 4. When to run them?
*   **Incidents**: SEV1/SEV2 outages (Required).
*   **Projects**: End of a major milestone (Retro).
*   **Wins**: "Success-mortems" to replicate what went right.

## Go Code Example: Incident Report Generator
This example generates a markdown-formatted Post-Mortem template based on incident data.

```go
package main

import (
	"fmt"
	"strings"
	"time"
)

type ActionItem struct {
	Owner       string
	Description string
	Type        string // Prevent, Detect, Mitigate
}

type PostMortem struct {
	Title       string
	Date        time.Time
	RootCause   string
	Impact      string
	FiveWhys    []string
	ActionItems []ActionItem
}

func (p PostMortem) GenerateReport() string {
	var b strings.Builder
	
	b.WriteString(fmt.Sprintf("# Post-Mortem: %s\n", p.Title))
	b.WriteString(fmt.Sprintf("**Date**: %s\n\n", p.Date.Format("2006-01-02")))
	
	b.WriteString("## Executive Summary\n")
	b.WriteString(fmt.Sprintf("%s\n\n", p.Impact))
	
	b.WriteString("## Root Cause Analysis (5 Whys)\n")
	for i, why := range p.FiveWhys {
		b.WriteString(fmt.Sprintf("%d. %s\n", i+1, why))
	}
	b.WriteString(fmt.Sprintf("\n**Root Cause**: %s\n\n", p.RootCause))
	
	b.WriteString("## Action Items\n")
	b.WriteString("| Type | Owner | Task |\n")
	b.WriteString("|---|---|---|\n")
	for _, item := range p.ActionItems {
		b.WriteString(fmt.Sprintf("| %s | %s | %s |\n", item.Type, item.Owner, item.Description))
	}
	
	return b.String()
}

func main() {
	pm := PostMortem{
		Title:  "Checkout Service Latency Spike",
		Date:   time.Now(),
		Impact: "Checkout latency increased by 500% for 20 mins. ~500 failed orders.",
		FiveWhys: []string{
			"Redis cache hit rate dropped to 0%.",
			"The Redis cluster was rebooting.",
			"An automated patch was applied during peak hours.",
			"The maintenance window was configured to UTC but interpreted as EST.",
			"Timezone configuration is inconsistent across infrastructure code.",
		},
		RootCause: "Timezone configuration mismatch in Terraform modules.",
		ActionItems: []ActionItem{
			{"DevOps", "Force UTC in all Terraform modules", "Prevent"},
			{"Backend", "Implement soft-fail if Cache is down", "Mitigate"},
			{"SRE", "Add alert for maintenance window active", "Detect"},
		},
	}

	fmt.Println(pm.GenerateReport())
}
```

## Interview Questions

### Q: "How do you run a post-mortem without it turning into a blame game?"
**A:**
*   **Set the stage**: Read the "Retrospective Prime Directive" at the start.
*   **Focus on 'How' not 'Who'**: Don't say "Why did YOU do that?"; say "How did the system allow that to happen?"
*   **Psychological Safety**: As the leader, admit your own contribution to the failure first to lower the temperature.

### Q: "What is the difference between a Root Cause and a Trigger?"
**A:**
*   **Trigger**: The immediate action that started the fire (e.g., "Deploying the config change").
*   **Root Cause**: The underlying defect that made the system flammable (e.g., "Lack of input validation on config files").
*   Fixing the trigger works once; fixing the root cause works forever.

### Q: "Why do we use the '5 Whys'?"
**A:**
*   To get past the superficial symptoms. Usually, the first answer is "Human Error," which is a dead end. By asking "Why" 5 times, we almost always land on a Process or Tooling failure that can be fixed.
