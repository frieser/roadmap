# Velocity Tracking

## Summary
Velocity is an Agile metric that measures the amount of work (usually in Story Points) a team completes during a Sprint. It is a tool for **Capacity Planning** (forecasting how much work can be done next), NOT a productivity metric for comparing teams.

## Detailed Explanation

### 1. What is Velocity?
*   **Definition**: The sum of story points for all *fully completed* user stories at the end of a sprint.
*   **Incomplete Work**: Does NOT count. Partial credit hides bottlenecks.
*   **The Trend**: A single sprint's velocity is noise. The *moving average* over 3-5 sprints is the signal.

### 2. Story Points vs. Hours
*   **Hours**: Estimate time (Absolute). Hard to predict due to interruptions.
*   **Points**: Estimate complexity/effort (Relative). "This task is twice as hard as that one."
*   **Fibonacci Sequence**: 1, 2, 3, 5, 8, 13. Used to reflect increasing uncertainty with size.

### 3. The "Yesterday's Weather" Pattern
To plan Sprint N, look at the velocity of Sprints N-1, N-2, and N-3. The best predictor of future performance is recent past performance.
*   *Formula*: `Capacity = Average(Last 3 Sprints) * Availability_Factor`

### 4. Anti-Patterns
*   **Velocity as a Target**: "We need to increase velocity by 10%." -> Team just inflates the numbers.
*   **Comparing Teams**: "Team A does 50 points, Team B does 20." -> Meaningless if Team B's "1 point" is bigger than Team A's.
*   **Individual Velocity**: Tracking points per person destroys teamwork.

## Go Code Example: Velocity Forecaster
This example calculates the moving average velocity and estimates how many sprints are needed to complete a backlog.

```go
package main

import (
	"fmt"
	"math"
)

type Sprint struct {
	Number int
	Points int // Points completed
}

type TeamStats struct {
	PastSprints []Sprint
	BacklogSize int // Total points remaining in backlog
}

// CalculateMovingAverage computes the average of the last N sprints
func (t TeamStats) CalculateMovingAverage(n int) float64 {
	if len(t.PastSprints) == 0 {
		return 0
	}
	
	sum := 0
	count := 0
	// Iterate backwards
	for i := len(t.PastSprints) - 1; i >= 0 && count < n; i-- {
		sum += t.PastSprints[i].Points
		count++
	}
	
	return float64(sum) / float64(count)
}

// EstimateCompletion forecasts timeline
func (t TeamStats) EstimateCompletion(avgVelocity float64) int {
	if avgVelocity == 0 {
		return -1 // Infinite
	}
	sprints := float64(t.BacklogSize) / avgVelocity
	return int(math.Ceil(sprints))
}

func main() {
	stats := TeamStats{
		PastSprints: []Sprint{
			{1, 28},
			{2, 32},
			{3, 25}, // Someone was sick
			{4, 35},
			{5, 30},
		},
		BacklogSize: 150,
	}

	// Use "Yesterday's Weather" (Last 3 sprints)
	avgVel := stats.CalculateMovingAverage(3) // (25 + 35 + 30) / 3 = 30
	
	fmt.Printf("Recent Velocity (Avg last 3): %.2f points/sprint\n", avgVel)
	
	sprintsNeeded := stats.EstimateCompletion(avgVel)
	fmt.Printf("Estimated Sprints to finish Backlog: %d\n", sprintsNeeded)
	
	// Volatility Check
	fmt.Println("\n--- Volatility Analysis ---")
	for _, s := range stats.PastSprints {
		diff := float64(s.Points) - avgVel
		fmt.Printf("Sprint %d: %d (Deviation: %+.1f)\n", s.Number, s.Points, diff)
	}
}
```

## Interview Questions

### Q: "Your team's velocity is erratic (20, 50, 15, 40). What do you do?"
**A:**
*   **Diagnose**: This indicates instability.
    *   Are stories too big? (Carry-over work creates "yoyo" effect).
    *   Are external blockers hitting mid-sprint?
    *   Is the definition of done (DoD) inconsistent?
*   **Action**: Focus on breaking stories down (invest in refinement) so they flow smoother.

### Q: "Management asks for a fixed delivery date for a large project. How do you use velocity to answer?"
**A:**
*   **Cone of Uncertainty**: "Based on our current velocity of X, we will finish between Date A (Optimistic) and Date B (Pessimistic)."
*   **Never give a single date**. Give a range with probability.

### Q: "Should we count bug fixes in velocity?"
**A:**
*   **Yes**: Fixing bugs takes time/effort. If you don't track it, your velocity looks artificially low, and you can't see how much capacity is being eaten by "failure demand" (fixing bad code) vs. "value demand" (new features).
