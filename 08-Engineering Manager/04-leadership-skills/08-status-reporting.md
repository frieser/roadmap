## Summary
Status reporting is the mechanism by which an Engineering Manager keeps stakeholders informed about project health, risks, and progress. Good reporting buys trust. It should be consistent, honest (no "watermelon" projects—green on outside, red on inside), and tailored to the audience (Execs want bullets, Devs want details).

## Detailed Explanation
### The RAG Status
*   **Red**: Critical issues, timeline will be missed, need help/decision immediately.
*   **Amber (Yellow)**: At risk, mitigation in place, might miss timeline.
*   **Green**: On track, no blockers.

### Key Components of a Report
1.  **Executive Summary**: TL;DR (Too Long; Didn't Read).
2.  **Current Progress**: What shipped?
3.  **Next Steps**: What's coming?
4.  **Risks/Blockers**: Where do we need help?

### Go Code Example: Adaptive Reporting
This code generates different versions of a status report based on the audience (Executive vs. Team).

```go
package leadership

import "fmt"

type ProjectStatus struct {
	Name        string
	RAG         string // Red, Amber, Green
	Details     []string
	Risks       []string
}

// GenerateReport tailors the output to the audience
func (p *ProjectStatus) GenerateReport(audience string) {
	fmt.Printf("--- Status Report: %s (%s) ---\n", p.Name, audience)
	
	switch audience {
	case "Executive":
		// High level, focus on RAG and Risks
		fmt.Printf("Status: %s\n", p.RAG)
		if p.RAG != "Green" {
			fmt.Println("Critical Risks:")
			for _, r := range p.Risks {
				fmt.Printf("- %s\n", r)
			}
		} else {
			fmt.Println("Project is on track.")
		}
		
	case "Team":
		// Detailed, focus on work items
		fmt.Printf("Status: %s\n", p.RAG)
		fmt.Println("Completed Items:")
		for _, d := range p.Details {
			fmt.Printf("- %s\n", d)
		}
		fmt.Println("Risks to Watch:")
		for _, r := range p.Risks {
			fmt.Printf("- %s\n", r)
		}
	}
	fmt.Println()
}

func main() {
	proj := ProjectStatus{
		Name:    "Migration to Cloud",
		RAG:     "Amber",
		Details: []string{"DB Replica created", "VPC set up", "Auth service migrated"},
		Risks:   []string{"Legacy firewall rules undefined", "Budget constraints"},
	}

	proj.GenerateReport("Executive")
	proj.GenerateReport("Team")
}
```

## Interview Questions
**Q: How do you report a significant project delay to leadership?**
**A:** I communicate it as early as possible (no surprises). I explain the *root cause* clearly (without blaming), the *impact* on the timeline, and the *options* we have (e.g., "We can ship on time if we cut feature X, or we can keep feature X and ship 2 weeks late"). I recommend a path forward and ask for their decision.

**Q: What is a 'Watermelon' status and how do you avoid it?**
**A:** A Watermelon status is Green on the outside (reporting "All Good") but Red on the inside (team is drowning, code is broken). It happens when managers are afraid to report bad news. I avoid it by creating a culture where "Amber" is seen as "Acting Responsibly" rather than "Failing." I reward early flagging of risks.

**Q: How do you handle stakeholders asking for status updates daily?**
**A:** Daily ad-hoc questions are disruptive. I establish a regular cadence (e.g., Weekly Email or Bi-weekly Demo) and push all updates to that channel. I might also offer a self-serve dashboard (Jira/Trello) where they can check status anytime without interrupting the team.
