# Tool Transitions

## Summary
Tool Transitions (e.g., Jira to Linear, Slack to Teams, GitHub to GitLab) are surprisingly emotional. Developers treat their tools as extensions of their bodies. A poor transition destroys productivity and morale. The key is **Data Migration** and **Training**.

## Detailed Explanation

### 1. The Migration Plan
*   **Audit**: What are we actually using? (You'll find 50% of Jira projects are dead).
*   **Mapping**: How does a Jira "Epic" map to a Linear "Project"?
*   **Cutover**: Pick a weekend. Read-only the old tool. Import data. Turn on new tool.

### 2. Dual Entry (The Danger Zone)
*   Avoid asking teams to update *both* tools. They will update neither.
*   Do a hard cutover for specific teams if a global cutover is too big.

### 3. Training
*   Don't just send a login link.
*   Host workshops. Record looms.
*   Create a #help-new-tool channel.

## Go Code Example: Data Mapper
Simulates mapping fields from an old tool to a new one.

```go
package main

import (
	"fmt"
)

type JiraTicket struct {
	Key     string
	Summary string
	Points  int
}

type LinearIssue struct {
	ID          string
	Title       string
	Estimate    int
	ImportedTag bool
}

func Migrate(j JiraTicket) LinearIssue {
	return LinearIssue{
		ID:          j.Key,     // Keep ID if possible
		Title:       j.Summary,
		Estimate:    j.Points,  // 1:1 mapping
		ImportedTag: true,
	}
}

func main() {
	oldTicket := JiraTicket{"PROJ-123", "Fix Login Bug", 3}
	newIssue := Migrate(oldTicket)

	fmt.Printf("Migrated %s -> %s (Points: %d)\n", oldTicket.Key, newIssue.Title, newIssue.Estimate)
}
```

## Interview Questions

### Q: "The team hates Jira and wants to move to Linear. Do you let them?"
**A:**
*   **Cost of Change**: Migration takes time. Is the friction high enough to justify the cost?
*   **Fragmentation**: If one team moves, does it break reporting for the Execs?
*   **Decision**: If the team is autonomous, yes. If they need to coordinate heavily with other Jira teams, no (or move everyone).

### Q: "How do you handle 'Tool Fatigue'?"
**A:**
*   Stop buying tools.
*   Consolidate. "We have Notion, Confluence, and Google Docs. Pick one."
*   Every new tool adds cognitive load.

### Q: "What is the most important part of a tool transition?"
**A:**
*   **Data Integrity**. If I lose my ticket history or comments, I can't do my job.
*   **Redirects**: Old links should redirect to the new tool if possible.
