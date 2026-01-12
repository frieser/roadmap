# Career Development Planning

## Summary
Career Development Planning is the intentional process of aligning an individual's personal aspirations with organizational needs. For an EM, it is the primary retention tool. It involves defining clear Career Ladders, co-creating Individual Development Plans (IDPs), and navigating the "Manager vs. Individual Contributor" pendulum.

## Detailed Explanation

### 1. The Career Ladder (Dual Track)
Modern tech organizations typically have two parallel tracks:
*   **Individual Contributor (IC)**: Junior -> Senior -> Staff -> Principal -> Distinguished. Focus is on technical depth and breadth.
*   **Management**: EM -> Director -> VP -> CTO. Focus is on people, process, and strategy.
*   *Key Principle*: It should be possible to reach equivalent pay/prestige on the IC track without becoming a manager.

### 2. The IDP (Individual Development Plan)
An IDP is a living document, revisited quarterly. It answers:
*   **Where am I now?** (Current skills, gaps).
*   **Where do I want to go?** (Next promotion, specific technology mastery).
*   **How will I get there?** (Specific projects, training, mentorship).

### 3. The 70-20-10 Model
*   **70% Experience**: Learning by doing (stretch assignments, tough projects).
*   **20% Exposure**: Learning from others (mentoring, code reviews, shadowing).
*   **10% Education**: Formal training (books, courses, conferences).

### 4. Promotion Packets
Promotions are lagging indicators of growth. The engineer acts at the next level for 3-6 months *before* getting the title. The "Packet" is the evidence portfolio proving this sustained performance.

## Go Code Example: Skill Gap Analysis
This example compares an engineer's current skills against the requirements for their target level (e.g., Senior Engineer) to generate an automated IDP suggestion.

```go
package main

import (
	"fmt"
)

// LevelRequirements defines what is needed for a role
type LevelRequirements struct {
	LevelName string
	Skills    map[string]int // Skill -> Required Proficiency (1-5)
}

// EngineerProfile represents the current state
type EngineerProfile struct {
	Name          string
	CurrentSkills map[string]int
}

// GapAnalysis generates a report
func AnalyzeGap(engineer EngineerProfile, target LevelRequirements) []string {
	var gaps []string

	for skill, reqScore := range target.Skills {
		currentScore, exists := engineer.CurrentSkills[skill]
		if !exists {
			currentScore = 0
		}

		if currentScore < reqScore {
			gap := fmt.Sprintf("[%s] Current: %d -> Need: %d", skill, currentScore, reqScore)
			gaps = append(gaps, gap)
		}
	}
	return gaps
}

func main() {
	// Target: Senior Engineer
	seniorReqs := LevelRequirements{
		LevelName: "Senior Engineer",
		Skills: map[string]int{
			"System Design":      4,
			"Coding":             5,
			"Mentorship":         3,
			"Project Management": 3,
		},
	}

	// Current: Junior Engineer
	alice := EngineerProfile{
		Name: "Alice",
		CurrentSkills: map[string]int{
			"System Design":      2,
			"Coding":             4,
			"Mentorship":         1,
			"Project Management": 1,
		},
	}

	fmt.Printf("Career Development Plan for %s (Target: %s)\n", alice.Name, seniorReqs.LevelName)
	fmt.Println("------------------------------------------------")
	
	gaps := AnalyzeGap(alice, seniorReqs)
	
	if len(gaps) == 0 {
		fmt.Println("✅ Ready for Promotion!")
	} else {
		fmt.Println("⚠️  Identified Gaps to Close:")
		for _, gap := range gaps {
			fmt.Println(gap)
		}
		
		fmt.Println("\nSuggested Actions:")
		fmt.Println("- Lead the design of a medium-sized feature (System Design)")
		fmt.Println("- Mentor a new hire or intern (Mentorship)")
		fmt.Println("- Manage the Jira board for one sprint (Project Management)")
	}
}
```

## Interview Questions

### Q: "How do you help an engineer decide between the IC and Manager tracks?"
**A:**
*   **The Pendulum**: Remind them it's not a one-way door.
*   **Motivation Check**: Do they get dopamine from *shipping code* (IC) or from *unblocking others* (Manager)?
*   **Trial Run**: Have them lead a project or manage an intern for 3 months. If they hate the meetings and people problems, they should stay IC.

### Q: "An engineer is frustrated they weren't promoted. How do you handle it?"
**A:**
*   **Validate**: Acknowledge their feelings.
*   **Gap Analysis**: Walk through the career ladder together. Show specifically where the gaps are (e.g., "Your code is Senior level, but your cross-team communication is still Junior level").
*   **Plan**: Co-create a concrete plan to close those gaps by the next cycle.

### Q: "What is the difference between Sponsorship and Mentorship?"
**A:**
*   **Mentorship**: "I will talk *to* you." (Advice, coaching, skill building).
*   **Sponsorship**: "I will talk *about* you." (Using political capital to get them into closed-door rooms, high-visibility projects, or promotion lists). Managers must be sponsors.
