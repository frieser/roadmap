## Summary
Team motivation is the art of creating an environment where engineers feel energized to do their best work. It moves beyond "carrots and sticks" (extrinsic motivation) to focus on intrinsic drivers. Daniel Pink's framework of **Autonomy, Mastery, and Purpose** is the gold standard for motivating knowledge workers.

## Detailed Explanation
### The Three Pillars (Daniel Pink)
1.  **Autonomy**: The desire to direct our own lives. (Let them choose the tools/approach).
2.  **Mastery**: The urge to get better and better at something that matters. (Challenge them, support learning).
3.  **Purpose**: The yearning to do what we do in the service of something larger than ourselves. (Connect code to user impact).

### Hygiene Factors (Herzberg)
Things that don't motivate but *demotivate* if missing:
*   Fair salary
*   Job security
*   Good tools/environment
*   Reasonable work hours

### Go Code Example: The Motivation Calculator
This code models an engineer's motivation as a function of intrinsic drivers, ensuring hygiene factors are met first.

```go
package leadership

import "fmt"

type Drivers struct {
	Autonomy float64 // 0.0 to 1.0
	Mastery  float64 // 0.0 to 1.0
	Purpose  float64 // 0.0 to 1.0
}

type Hygiene struct {
	SalaryFairness float64 // 0.0 to 1.0
	PsychSafety    bool
}

type Engineer struct {
	Name    string
	Drivers Drivers
	Hygiene Hygiene
}

// CalculateMotivation returns a score from 0-100
func (e *Engineer) CalculateMotivation() float64 {
	// 1. Check Hygiene Factors (The Foundation)
	if !e.Hygiene.PsychSafety {
		return 0.0 // Fear kills motivation instantly
	}
	if e.Hygiene.SalaryFairness < 0.5 {
		return 20.0 // Low salary distracts from work
	}

	// 2. Calculate Intrinsic Motivation
	// Weighted average: Autonomy + Mastery + Purpose
	score := (e.Drivers.Autonomy + e.Drivers.Mastery + e.Drivers.Purpose) / 3.0
	return score * 100
}

func main() {
	alice := Engineer{
		Name: "Alice",
		Hygiene: Hygiene{SalaryFairness: 0.9, PsychSafety: true},
		Drivers: Drivers{
			Autonomy: 0.8, // Loves freedom
			Mastery:  0.9, // Loves learning Go
			Purpose:  0.4, // Unclear on product vision
		},
	}

	fmt.Printf("Motivation Score for %s: %.1f%%\n", alice.Name, alice.CalculateMotivation())
	fmt.Println("Action: Alice needs to see the customer impact (Purpose) to reach 100%.")
}
```

## Interview Questions
**Q: How do you motivate a team that is burnt out?**
**A:** First, I stop the bleeding. We look at the workload and cut non-essential tasks (scope hammer). I focus on "Hygiene Factors"—ensuring they take time off and rest. Once stable, I re-introduce motivation by celebrating small wins to rebuild confidence (Mastery) and reconnecting them to *why* their work matters (Purpose), but only after they are rested.

**Q: How do you motivate engineers to do "boring" maintenance work?**
**A:** I frame it through Mastery and Purpose. "Refactoring this legacy code will make us faster (Mastery) and stop the pager from waking us up at 3 AM (Autonomy/Quality of Life)." I also ensure the work is shared fairly so no single person is stuck with the grunt work.

**Q: What is the difference between intrinsic and extrinsic motivation?**
**A:** Extrinsic motivation is external (bonuses, titles, fear of firing). It works for repetitive tasks but kills creativity. Intrinsic motivation comes from within (curiosity, pride in craft). For software engineering, intrinsic motivation is far more powerful and sustainable.
