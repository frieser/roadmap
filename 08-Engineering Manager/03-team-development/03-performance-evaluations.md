# Performance Evaluations

## Summary
Performance Evaluations are the formal mechanism for aligning an engineer's work with the company's expectations. While feedback should be continuous (weekly 1:1s), the evaluation cycle (reviews) is where calibration, compensation, and promotion decisions happen. A good evaluation system is fair, transparent, and evidence-based, avoiding "recency bias."

## Detailed Explanation

### 1. The Cycle
*   **Goal Setting (Beginning)**: Defining clear OKRs (Objectives and Key Results) or KPIs. "What does success look like?"
*   **Continuous Feedback (Middle)**: No surprises. If someone is failing, they should know months before the review.
*   **Self-Reflection (End)**: The engineer writes their own brag doc.
*   **Peer Reviews (360)**: Gathering input from teammates, PMs, and other stakeholders.
*   **Manager Assessment**: Synthesizing all data into a rating.

### 2. Biases to Watch Out For
*   **Recency Bias**: Judging only the last month of work instead of the full 6 months.
*   **Halo Effect**: Letting being "good at one thing" (e.g., confident speaker) overshadow poor technical work.
*   **Central Tendency**: Rating everyone as "Meets Expectations" to avoid conflict.

### 3. The 9-Box Grid
A common tool for talent calibration:
*   **X-Axis**: Performance (Results).
*   **Y-Axis**: Potential (Growth/Leadership).
*   **High/High**: Star performer (Promote/Retain).
*   **Low/Low**: Underperformer (PIP/Exit).

### 4. Handling Underperformance
*   **PIP (Performance Improvement Plan)**: A formal document outlining deficiencies, expected results, and a timeline (usually 30-60 days). It is a final effort to save the employee, but often serves as documentation for termination.

## Go Code Example: Performance Calibration Tool
This example aggregates 360-degree feedback and calculates a performance score to help a manager calibrate their team.

```go
package main

import (
	"fmt"
)

type Rating int

const (
	NeedsImprovement Rating = 1
	MeetsExpectations Rating = 2
	ExceedsExpectations Rating = 3
	Superstar Rating = 4
)

type Employee struct {
	Name        string
	SelfReview  Rating
	ManagerReview Rating
	PeerReviews []Rating
}

// CalculateFinalScore computes a weighted score
// Manager: 50%, Peers: 30%, Self: 20% (Self-awareness check)
func (e Employee) CalculateFinalScore() float64 {
	peerSum := 0
	for _, r := range e.PeerReviews {
		peerSum += int(r)
	}
	peerAvg := float64(peerSum) / float64(len(e.PeerReviews))

	weightedScore := (float64(e.ManagerReview) * 0.50) + 
					 (peerAvg * 0.30) + 
					 (float64(e.SelfReview) * 0.20)
	
	return weightedScore
}

func main() {
	// Scenario: Calibrating "John"
	john := Employee{
		Name:          "John Doe",
		SelfReview:    ExceedsExpectations, // He thinks he's great (3)
		ManagerReview: MeetsExpectations,   // Manager thinks he's okay (2)
		PeerReviews:   []Rating{NeedsImprovement, MeetsExpectations, MeetsExpectations}, // Peers are mixed (~1.6)
	}

	score := john.CalculateFinalScore()
	fmt.Printf("Performance Score for %s: %.2f\n", john.Name, score)
	// (2 * 0.5) + (1.66 * 0.3) + (3 * 0.2) = 1.0 + 0.5 + 0.6 = 2.1

	if score < 1.5 {
		fmt.Println("Action: Performance Improvement Plan (PIP)")
	} else if score > 3.5 {
		fmt.Println("Action: Promote")
	} else {
		fmt.Println("Action: Retain & Coach")
	}
	
	// Calibration check: Gap between Self and Manager
	if john.SelfReview > john.ManagerReview+1 {
		fmt.Println("⚠️ Alert: Self-awareness gap detected. Discuss expectations.")
	}
}
```

## Interview Questions

### Q: "How do you handle a high performer who is toxic to the team?"
**A:**
*   **Address Immediately**: "The standard you walk past is the standard you accept."
*   **Separate Results from Behavior**: Acknowledge the code is good, but emphasize that teamwork is a core job requirement.
*   **Ultimatum**: If behavior doesn't change, they must go. A toxic rockstar drives away other good engineers.

### Q: "Describe how you prepare for a difficult performance review."
**A:**
*   **Gather Data**: Don't rely on memory. Pull commit logs, peer feedback, and past 1:1 notes.
*   **Script the Opening**: The first 2 minutes set the tone. Be direct.
*   **Focus on Impact**: Not "You are lazy," but "Missing the deadline caused the marketing launch to slip."

### Q: "What is your philosophy on PIPs? Are they just a formality for firing?"
**A:**
*   **Ideally No**: A PIP *should* be a genuine roadmap to recovery.
*   **Realistically**: Statistics show ~50-70% of PIPs result in exit.
*   **My Approach**: I fight hard *before* the PIP (coaching). Once on a PIP, I support them 100%, but I also advise them to start looking at options so they land on their feet if it doesn't work out.
