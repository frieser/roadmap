# Strategic Proposals

## Summary
Strategic Proposals are formal documents pitched to leadership to initiate a major change (e.g., "Rewrite the Monolith," "Open a London Office," "Acquire a Competitor"). Unlike a Jira ticket, a proposal requires a comprehensive view of strategy, finance, and operations.

## Detailed Explanation

### 1. The 6-Pager (Amazon Style)
Many tech companies use narratives instead of PowerPoint.
1.  **Introduction**: The problem and opportunity.
2.  **Goals**: Success metrics.
3.  **Tenets**: Guiding principles (e.g., "Speed over Cost").
4.  **State of the Business**: Current data.
5.  **The Plan**: Phases, milestones, resources.
6.  **FAQ**: Anticipating objections.

### 2. Disagree and Commit
*   The goal of a proposal review is not consensus (everyone agrees).
*   The goal is **clarity**.
*   Once the decision is made (even if it's "No"), the EM must support it fully to the team.

### 3. Horizon Planning
*   **Horizon 1**: Core business (Optimize now).
*   **Horizon 2**: Emerging opportunities (Scale next).
*   **Horizon 3**: Transformative bets (Research for future).
*   *Ensure your proposal fits the right horizon.*

## Go Code Example: Proposal Voting System
This example simulates a decision-making process where stakeholders vote on a strategic proposal.

```go
package main

import (
	"fmt"
)

type Vote string

const (
	Yes           Vote = "YES"
	No            Vote = "NO"
	DisagreeCommit Vote = "DISAGREE_AND_COMMIT"
)

type Stakeholder struct {
	Name string
	Role string
	Vote Vote
}

func TallyVotes(proposalName string, voters []Stakeholder) {
	fmt.Printf("--- Decision Meeting: %s ---\n", proposalName)
	
	yesCount := 0
	commitCount := 0
	
	for _, v := range voters {
		fmt.Printf("%s (%s): %s\n", v.Name, v.Role, v.Vote)
		if v.Vote == Yes || v.Vote == DisagreeCommit {
			yesCount++
		}
		if v.Vote == DisagreeCommit {
			commitCount++
		}
	}

	fmt.Println("--- Result ---")
	if yesCount > len(voters)/2 {
		fmt.Println("✅ APPROVED")
		if commitCount > 0 {
			fmt.Println("(Note: Some stakeholders disagreed but committed. Manage expectations.)")
		}
	} else {
		fmt.Println("❌ REJECTED")
	}
}

func main() {
	voters := []Stakeholder{
		{"CEO", "Exec", Yes},
		{"CFO", "Exec", No}, // Too expensive
		{"CTO", "Exec", DisagreeCommit}, // Hates the tech, but agrees we need to move
		{"VP Sales", "Exec", Yes},
	}

	TallyVotes("Project: Move to Cloud", voters)
}
```

## Interview Questions

### Q: "How do you write a proposal that gets approved?"
**A:**
*   **Pre-wire**: Meet with key stakeholders *before* the official meeting. Incorporate their feedback. A proposal should rarely be a surprise in the final meeting.
*   **Align with Goals**: "This proposal supports the Company OKR of 'International Expansion'."
*   **Option Value**: Offer 3 options (Do nothing, Gold Plated, Recommended). This makes the Recommended option look balanced.

### Q: "What do you do if your proposal is rejected?"
**A:**
*   **Don't take it personally**.
*   **Ask why**: Is it timing? Budget? Strategy mismatch?
*   **Archive it**: Keep the doc. Conditions change. In 6 months, it might be the right idea.

### Q: "Why use written narratives (6-pagers) instead of slides?"
**A:**
*   Slides are low-information density and allow "hand-waving."
*   Writing forces clarity of thought. It exposes gaps in logic that bullet points hide.
*   It levels the playing field for introverts vs. charismatic speakers.
