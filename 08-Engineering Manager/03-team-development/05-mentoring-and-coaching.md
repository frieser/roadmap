# Mentoring and Coaching

## Summary
Mentoring and Coaching are the twin engines of team growth. While often used interchangeably, they are distinct modes. **Mentoring** is directive ("I've been there, do this"), filling a knowledge gap. **Coaching** is non-directive ("What do you think?"), building problem-solving muscle. A great EM switches fluently between these modes based on the context (Situational Leadership).

## Detailed Explanation

### 1. Mentoring (The "Sage")
*   **Goal**: Transfer knowledge or skill.
*   **Context**: The report is a junior, new to the company, or learning a specific technology you know well.
*   **Mechanism**: Showing patterns, sharing war stories, code reviews, "Watch me do this."

### 2. Coaching (The "Guide")
*   **Goal**: Unlock potential and critical thinking.
*   **Context**: The report is senior, facing an ambiguous problem, or needs to own the solution.
*   **Mechanism**: The **GROW Model**:
    *   **G**oal: What do you want to achieve?
    *   **R**eality: What is happening now?
    *   **O**ptions: What could you do?
    *   **W**ill: What *will* you do?

### 3. Situational Leadership (Hersey-Blanchard)
*   **S1 Directing**: Low Competence, High Commitment -> Mentoring/Instruction.
*   **S2 Coaching**: Low Competence, Low Commitment -> Selling/Encouraging.
*   **S3 Supporting**: High Competence, Low Confidence -> Listening/Facilitating.
*   **S4 Delegating**: High Competence, High Commitment -> Autonomy.

### 4. Setting up a Mentorship Program
*   **Pairing**: Match seniors with juniors (not just for code, but for design).
*   **Duration**: Set a time limit (e.g., 6 months) so it doesn't feel like a life sentence.
*   **Goals**: Define what "success" looks like at the start.

## Go Code Example: Mentorship Matching Algorithm
This example simulates a simple stable marriage-style matching algorithm to pair Mentors with Mentees based on skills they want to teach vs. learn.

```go
package main

import (
	"fmt"
)

type Engineer struct {
	Name        string
	CanTeach    []string
	WantsToLearn []string
	MatchedWith string
}

func Match(mentors []*Engineer, mentees []*Engineer) {
	fmt.Println("--- Mentorship Matching Results ---")
	
	for _, mentee := range mentees {
		bestMatch := ""
		maxOverlap := 0

		for _, mentor := range mentors {
			if mentor.MatchedWith != "" {
				continue // Mentor already taken (simplified)
			}
			
			score := calculateOverlap(mentor.CanTeach, mentee.WantsToLearn)
			if score > maxOverlap {
				maxOverlap = score
				bestMatch = mentor.Name
			}
		}

		if bestMatch != "" {
			mentee.MatchedWith = bestMatch
			// Mark mentor as taken
			for _, m := range mentors {
				if m.Name == bestMatch {
					m.MatchedWith = mentee.Name
				}
			}
			fmt.Printf("✅ %s matched with Mentor %s (Overlap: %d skills)\n", mentee.Name, bestMatch, maxOverlap)
		} else {
			fmt.Printf("⚠️  No suitable mentor found for %s\n", mentee.Name)
		}
	}
}

func calculateOverlap(teach []string, learn []string) int {
	count := 0
	for _, t := range teach {
		for _, l := range learn {
			if t == l {
				count++
			}
		}
	}
	return count
}

func main() {
	mentors := []*Engineer{
		{Name: "Senior Sam", CanTeach: []string{"Go", "Kubernetes", "System Design"}},
		{Name: "Staff Sarah", CanTeach: []string{"Leadership", "Politics", "Architecture"}},
	}

	mentees := []*Engineer{
		{Name: "Junior Jim", WantsToLearn: []string{"Go", "System Design"}},
		{Name: "Senior Steve", WantsToLearn: []string{"Leadership", "Politics"}},
	}

	Match(mentors, mentees)
}
```

## Interview Questions

### Q: "When do you step in to solve a problem vs. letting the team struggle?"
**A:**
*   **The Struggle is the Point**: If the cost of failure is low (e.g., a dev environment bug), let them struggle. It builds muscle.
*   **The Cliff**: If the cost of failure is high (e.g., production outage, missing a critical deadline), step in to *Direct*.
*   **My Rule**: I ask "Is this a learning opportunity or a business risk?"

### Q: "How do you coach a Senior Engineer who thinks they know everything?"
**A:**
*   **Find a bigger pond**: Their arrogance often comes from being the smartest in the *current* room.
*   **Strategy**: Assign them a problem they *can't* solve alone (e.g., cross-team negotiation).
*   **Feedback**: "Your technical skills are L5, but your collaboration is L3. To get to Staff, you need to influence people who don't report to you."

### Q: "Tell me about a time you mentored someone to a promotion."
**A:**
*   **Focus**: Identification of the gap, the specific plan (IDP), the regular check-ins, and the successful outcome (advocacy in the calibration meeting).
