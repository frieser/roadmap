# Hiring and Recruitment

## Summary
Hiring and recruitment is one of the highest-leverage activities for an Engineering Manager. It involves not just filling seats, but designing a process that is fair, rigorous, and aligned with team culture. A great process minimizes bias, ensures a positive candidate experience (even for those rejected), and consistently identifies engineers who raise the bar.

## Detailed Explanation

### 1. The Funnel Approach
Recruiting is a sales funnel. You must track metrics at each stage:
- **Sourcing**: Outbound (LinkedIn Recruiter) vs. Inbound (Applications) vs. Referrals (Highest conversion).
- **Screening**: Recruiter phone screen (culture/logistics) -> Technical Screen (basic competence).
- **Onsite/Loop**: Deep dive into coding, system design, and behavioral traits.
- **Offer**: Closing the candidate.

### 2. Structured Interviewing
To reduce bias, use **Structured Interviews**:
- **Rubrics**: Every question must have a predefined "Good," "Ok," and "Bad" answer key.
- **Role Assignment**: Assign specific competencies to interviewers (e.g., "Interviewer A: Architecture," "Interviewer B: Collaboration").
- **Calibration**: Debrief sessions where the team discusses evidence, not feelings.

### 3. Hiring for "Culture Add," Not "Culture Fit"
"Culture fit" often leads to hiring people who look and think like the existing team (homogeneity). "Culture add" looks for missing perspectives that make the team stronger (e.g., "We need someone who is more process-oriented" or "We need a devil's advocate").

### 4. The Bar Raiser
A concept from Amazon: one interviewer (outside the immediate hiring team) has veto power to ensure the candidate is better than 50% of the current team in that role.

## Go Code Example: Candidate Scoring System
This example models a structured interview scoring system. It aggregates scores from multiple interviewers and determines if a candidate passes the "Bar" based on a weighted average.

```go
package main

import (
	"fmt"
)

// ScoreType defines the rating scale
type ScoreType int

const (
	StrongNo   ScoreType = 1
	No         ScoreType = 2
	Yes        ScoreType = 3
	StrongYes  ScoreType = 4
)

// Feedback represents one interviewer's evaluation
type Feedback struct {
	Interviewer string
	Competency  string    // e.g., "System Design", "Coding"
	Score       ScoreType
	Notes       string
}

// Candidate represents an applicant
type Candidate struct {
	Name     string
	Feedbacks []Feedback
}

// HiringDecision logic
func (c *Candidate) Evaluate() string {
	totalScore := 0.0
	strongYesCount := 0
	strongNoCount := 0

	for _, f := range c.Feedbacks {
		totalScore += float64(f.Score)
		if f.Score == StrongYes {
			strongYesCount++
		}
		if f.Score == StrongNo {
			strongNoCount++
		}
	}

	average := totalScore / float64(len(c.Feedbacks))
	fmt.Printf("Candidate: %s | Avg Score: %.2f\n", c.Name, average)

	// Decision Logic
	if strongNoCount > 0 {
		return "REJECT (Vetoed)"
	}
	if average >= 3.0 && strongYesCount >= 1 {
		return "HIRE"
	}
	return "REJECT (Does not meet bar)"
}

func main() {
	candidate := Candidate{
		Name: "Alice Engineer",
		Feedbacks: []Feedback{
			{"Dave", "System Design", Yes, "Good grasp of caching"},
			{"Sarah", "Coding", StrongYes, "Optimal solution, great tests"},
			{"Jim", "Culture Add", Yes, "Good communication"},
			{"Bar Raiser", "Overall", Yes, "Solid senior engineer"},
		},
	}

	decision := candidate.Evaluate()
	fmt.Println("Final Decision:", decision)
	
	// Example of a bad candidate
	badCandidate := Candidate{
		Name: "Bob Developer",
		Feedbacks: []Feedback{
			{"Dave", "System Design", No, "Struggled with databases"},
			{"Sarah", "Coding", StrongNo, "Code did not compile"},
		},
	}
	fmt.Println("Final Decision:", badCandidate.Evaluate())
}
```

## Interview Questions

### Q: "How do you mitigate unconscious bias in your hiring process?"
**A:**
*   **Anonymization**: Removing names/photos from resumes where possible.
*   **Structured Rubrics**: Grading against a standard, not comparing candidates to each other.
*   **Diverse Panels**: Ensuring the interview loop includes people from different backgrounds/genders.

### Q: "A candidate is technically brilliant but was rude to the receptionist. Do you hire them?"
**A:**
*   **No**. This is a "Brilliant Jerk."
*   **Reasoning**: Their toxic behavior will destroy team morale and productivity, costing far more than their technical output is worth. Values are non-negotiable.

### Q: "How do you handle a disagreement between two interviewers about a candidate?"
**A:**
*   **Data Mining**: Ask for specific examples of behavior observed (e.g., "When you say they were arrogant, what exactly did they say?").
*   **Weighing Signals**: If one interviewer probed a specific area deeper, their signal carries more weight.
*   **Reference Checks**: Use references to validate the specific concern.
