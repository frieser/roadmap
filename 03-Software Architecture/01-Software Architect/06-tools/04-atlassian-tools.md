---
---

## Summary
Atlassian provides a suite of tools that are the industry standard for managing the Software Development Life Cycle (SDLC) in many enterprises. **Jira** (Project Management), **Confluence** (Documentation), and **Bitbucket** (Source Control/CI) integrate to provide traceability from a "Business Idea" to "Code Deployment."

## Detailed Explanation

### 1. Jira (Tracking Work)
*   **Issues**: Everything is an issue (Story, Bug, Task, Epic).
*   **Agile Boards**: Kanban or Scrum boards to visualize Work In Progress (WIP).
*   **Workflows**: Define the lifecycle (To Do -> In Progress -> Code Review -> Done). Architects often enforce steps like "Architecture Review" for large Epics.

### 2. Confluence (Knowledge Base)
*   **Collaboration**: Where requirements, ADRs (Architectural Decision Records), and Post-Mortems live.
*   **Integration**: You can link a Jira ticket to a Confluence page ("Here are the specs for this ticket").

### 3. Bitbucket (Code)
*   **Git Repositories**: Source control.
*   **Pipelines**: Integrated CI/CD.
*   **Code Review**: Pull Requests integrated with Jira (the ticket updates status when PR is merged).

## Go-Specific Context/Examples

While Go developers often prefer lightweight tools (GitHub/GitLab), you often integrate Go tools with Jira automation.

### Example: Jira Webhook Handler in Go
Automating workflows: "When a ticket is moved to 'Deploy', trigger a Go service."

```go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
)

type JiraWebhook struct {
	Issue struct {
		Key    string `json:"key"`
		Fields struct {
			Summary string `json:"summary"`
		} `json:"fields"`
	} `json:"issue"`
}

func jiraHandler(w http.ResponseWriter, r *http.Request) {
	var event JiraWebhook
	if err := json.NewDecoder(r.Body).Decode(&event); err != nil {
		http.Error(w, "Bad Request", 400)
		return
	}
	
	fmt.Printf("Received update for Issue: %s - %s\n", event.Issue.Key, event.Issue.Fields.Summary)
	// Trigger deployment logic, update status, etc.
}

func main() {
	http.HandleFunc("/webhook", jiraHandler)
	http.ListenAndServe(":8080", nil)
}
```

## Interview Questions

**Q: How do you trace a production bug back to the requirements?**
**A:** Using the Atlassian suite: The Deployment pipeline (Bitbucket) links the Commit to the Jira Ticket. The Jira Ticket links to the Confluence Requirement page. This "Golden Thread" allows you to see *who* wrote the code, *why* (the ticket), and *what* was expected (the doc).

**Q: Should architecture diagrams live in Confluence or the Code (Readme)?**
**A:** Both, but for different audiences. High-level, logical architecture (Business capabilities) belongs in **Confluence** for stakeholders. Low-level, physical architecture (Package structure, API specs) belongs in the **Repo (README/Markdown)** so it stays in sync with the code version.

**Q: How can you use Jira to manage Technical Debt?**
**A:** Create a dedicated "Tech Debt" backlog or a specific label/issue type in Jira. Allocate a fixed capacity (e.g., 20%) of every sprint to tackle these tickets. If it's not in Jira, it doesn't exist and won't get prioritized.
