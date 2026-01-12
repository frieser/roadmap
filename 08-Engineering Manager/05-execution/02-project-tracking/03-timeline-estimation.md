## Summary
Timeline Estimation is the practice of predicting the duration required to complete a task or project. Accurate estimation is notoriously difficult in software engineering due to unknowns and complexity. Techniques range from rough order of magnitude (ROM) to detailed bottom-up analysis.

## Detailed Explanation
Estimates are probabilistic, not deterministic. A single date usually implies 100% certainty, which is rarely true.

### Common Techniques
1.  **PERT (Program Evaluation and Review Technique)**: Uses three values: Optimistic (O), Most Likely (M), and Pessimistic (P).
    *   *Weighted Average = (O + 4M + P) / 6*
2.  **Planning Poker**: Agile teams vote on complexity (Story Points) using Fibonacci numbers to reach consensus and uncover hidden assumptions.
3.  **T-Shirt Sizing**: High-level categorization (XS, S, M, L, XL) for roadmap planning.
4.  **Cone of Uncertainty**: Estimates become more accurate as the project progresses and details solidify.

### Hofstadter's Law
"It always takes longer than you expect, even when you take into account Hofstadter's Law."

## Go Code Example
Implementing the PERT estimation formula and a Monte Carlo simulation to forecast project completion probability.

```go
package main

import (
	"fmt"
	"math"
	"math/rand"
	"time"
)

type Task struct {
	Name       string
	Optimistic float64
	MostLikely float64
	Pessimistic float64
}

// PERT returns the weighted average and standard deviation
func (t Task) PERT() (float64, float64) {
	mean := (t.Optimistic + 4*t.MostLikely + t.Pessimistic) / 6
	stdDev := (t.Pessimistic - t.Optimistic) / 6
	return mean, stdDev
}

// RunMonteCarloSim simulates the project N times to find probability of finishing by 'target'
func RunMonteCarloSim(tasks []Task, target float64, simulations int) {
	successCount := 0
	rand.Seed(time.Now().UnixNano())

	for i := 0; i < simulations; i++ {
		totalTime := 0.0
		for _, task := range tasks {
			// Triangular distribution approximation
			mean, stdDev := task.PERT()
			// Generate random duration based on normal distribution around PERT mean
			duration := rand.NormFloat64()*stdDev + mean
			totalTime += duration
		}

		if totalTime <= target {
			successCount++
		}
	}

	probability := (float64(successCount) / float64(simulations)) * 100
	fmt.Printf("Probability of finishing in %.1f days: %.2f%%\n", target, probability)
}

func main() {
	tasks := []Task{
		{"DB Setup", 1, 2, 4},
		{"API Dev", 3, 5, 9},
		{"Frontend", 2, 4, 8},
	}

	var totalMean float64
	for _, t := range tasks {
		mean, _ := t.PERT()
		totalMean += mean
	}
	fmt.Printf("PERT Weighted Average Duration: %.2f days\n", totalMean)

	// Will we finish in 12 days?
	RunMonteCarloSim(tasks, 12.0, 10000)
}
```

## Interview Questions
**Q: Why do software estimates often fail?**
**A:** They fail due to the "Unknown Unknowns," cognitive biases like the Planning Fallacy (optimism bias), context switching costs, and failure to account for non-coding time (meetings, code review, deployment issues).

**Q: How do you estimate a task you have never done before?**
**A:** I use a "Spike" (a time-boxed investigation task) to learn enough to provide an estimate. Alternatively, I break the task down until the sub-components are recognizable, or use comparative estimation (referencing a similar past task).

**Q: Explain Story Points vs. Hours.**
**A:** Hours measure time (absolute). Story Points measure complexity, risk, and effort relative to other tasks (relative). Points account for the fact that a senior dev might take 1 hour and a junior 5 hours for the same task, but the *complexity* is constant.
