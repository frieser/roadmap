# Vision Alignment

## Summary
Vision Alignment is the act of connecting the day-to-day work of engineers (Jira tickets) to the multi-year mission of the company (Vision). Without alignment, teams work hard but drift in different directions. EMs must be "Chief Repeaters," constantly reiterating the Vision to ensure every decision pulls the same rope.

## Detailed Explanation

### 1. Mission vs. Vision vs. Strategy
*   **Mission**: Why we exist. (e.g., "To organize the world's information"). *Permanent.*
*   **Vision**: What the future looks like if we succeed. (e.g., "A computer in every home"). *10-year goal.*
*   **Strategy**: How we get there. (e.g., "Move to Mobile First"). *1-3 year plan.*
*   **Tactics**: What we do today. (e.g., "Fix bug #123").

### 2. The "Cascade"
*   CEO sets Vision -> VP sets Strategy -> EM sets Execution Plan -> Dev writes Code.
*   *Broken Cascade*: Dev writes code that doesn't support the Strategy. This is "misalignment."

### 3. Communicating the "Why"
*   Engineers are problem solvers. If you give them the "What" (Build a bridge) without the "Why" (To cross the river), they might build a beautiful bridge in the wrong place.
*   Always start meetings with the context.

## Go Code Example: Alignment Vector Check
This conceptual code calculates if a team's tasks are aligned with the company vector.

```go
package main

import (
	"fmt"
	"math"
)

// Vector represents direction (simplified 2D)
type Vector struct {
	X, Y float64
}

func (v Vector) DotProduct(other Vector) float64 {
	return v.X*other.X + v.Y*other.Y
}

func main() {
	companyVision := Vector{X: 1.0, Y: 0.0} // Moving East (Growth)
	
	teamTasks := []struct {
		Name   string
		Vector Vector
	}{
		{"Feature A (Growth)", Vector{1.0, 0.1}},    // Aligned
		{"Refactor B (Quality)", Vector{0.5, 0.5}},  // Partially aligned
		{"Side Project C", Vector{-1.0, 0.0}},       // Misaligned (Moving West)
	}

	fmt.Println("--- Alignment Check ---")
	for _, t := range teamTasks {
		alignment := companyVision.DotProduct(t.Vector)
		
		status := ""
		if alignment > 0.8 {
			status = "✅ Highly Aligned"
		} else if alignment > 0 {
			status = "⚠️  Loose Alignment"
		} else {
			status = "❌ Misaligned (Wasted Effort)"
		}

		fmt.Printf("Task: %s | Score: %.2f | %s\n", t.Name, alignment, status)
	}
}
```

## Interview Questions

### Q: "The CEO changes the vision every 6 months. How do you handle this 'Whiplash'?"
**A:**
*   **Buffer**: I don't pass every ripple down to the team immediately. I wait to see if it sticks.
*   **Abstract**: I architect the system to be flexible (modular) so strategy shifts don't require total rewrites.
*   **Feedback**: I tell the CEO, "Changing direction costs us 2 months of velocity. Are you sure?"

### Q: "How do you measure alignment?"
**A:**
*   **The 'Elevator Test'**: Ask a random engineer, "Why are we working on this project?"
*   If they answer with the business goal ("To increase retention"), we are aligned.
*   If they answer with the technical task ("To upgrade React"), we are loosely aligned.
*   If they say "I don't know," we are misaligned.

### Q: "How often should you repeat the vision?"
**A:**
*   **Constant Repetition**: Humans need to hear something ~7 times to believe it.
*   Include it in All-Hands, Sprint Planning, and 1:1s.
