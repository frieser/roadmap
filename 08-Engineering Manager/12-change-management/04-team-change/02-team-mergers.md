# Team Mergers

## Summary
Team Mergers involve combining two distinct teams into one unit. This often happens after an acquisition or a consolidation strategy. The challenge is merging two different *cultures*, *processes*, and *tech stacks* without causing a civil war.

## Detailed Explanation

### 1. The "Us vs. Them" Dynamic
*   Team A thinks their code is clean and Team B's is trash.
*   Team B thinks Team A is slow and bureaucratic.
*   *Goal*: Create a new identity (Team C). Kill the old names.

### 2. Process Unification
*   **Scrum vs. Kanban**: Pick one. Don't run two systems.
*   **Jira vs. Trello**: Migrate to one tool immediately. Fragmented tools = fragmented communication.

### 3. The "Ambassador" Pattern
*   Seed the new team with culturally strong members from both sides.
*   Have them pair program (cross-pollination) to break down stereotypes.

## Go Code Example: Skill Integration Analysis
This tool checks for redundant roles or skill gaps when merging two rosters.

```go
package main

import (
	"fmt"
)

type Employee struct {
	Name string
	Role string
}

func MergeRosters(team1, team2 []Employee) map[string]int {
	roleCounts := make(map[string]int)
	
	for _, e := range team1 {
		roleCounts[e.Role]++
	}
	for _, e := range team2 {
		roleCounts[e.Role]++
	}
	
	return roleCounts
}

func main() {
	teamAcquired := []Employee{{"Dev1", "Backend"}, {"Dev2", "Backend"}, {"PM1", "Product"}}
	teamHost := []Employee{{"Dev3", "Frontend"}, {"Dev4", "Backend"}, {"PM2", "Product"}}

	counts := MergeRosters(teamAcquired, teamHost)

	fmt.Println("--- Merged Team Composition ---")
	for role, count := range counts {
		fmt.Printf("%s: %d\n", role, count)
		if role == "Product" && count > 1 {
			fmt.Println("⚠️  Warning: Duplicate Product Owners. Clarify ownership.")
		}
	}
}
```

## Interview Questions

### Q: "You are merging a Startup team (Speed) with an Enterprise team (Stability). How do you handle the culture clash?"
**A:**
*   **Acknowledge Strengths**: "We need Startup speed for features, but Enterprise stability for core."
*   **Define New Norms**: Create a team charter that explicitly states the compromise. "We move fast on UI, but require 100% test coverage on Billing."
*   **Socialize**: Forced bonding (offsite) helps humanize the "other side."

### Q: "What do you do with duplicate roles (e.g., 2 Tech Leads)?"
**A:**
*   **Split Scope**: One becomes "Architect" (Technical breadth), one becomes "Team Lead" (Delivery focus).
*   **Split Teams**: If the merged team is too big (>10), split it again along different lines so both can lead.
*   **Honesty**: If there is truly only one seat, the most qualified gets it, and the other must step down or move.

### Q: "How do you prevent the 'Acquired' team from quitting?"
**A:**
*   **Autonomy**: Don't crush their process on Day 1. Give them time to adapt.
*   **Respect**: Praise their legacy. "You built a $10M business with this code. Respect."
*   **Golden Handcuffs**: Retention bonuses help, but culture keeps them long-term.
