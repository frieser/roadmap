## Summary
Resource Allocation is the strategic assignment of available resources (people, budget, equipment) to tasks in a way that maximizes efficiency and ensures project delivery. In engineering, this primarily means assigning the right engineers to the right tasks based on skills, availability, and career growth interests.

## Detailed Explanation
Effective resource allocation balances business needs with team health. Over-allocation leads to burnout; under-allocation leads to boredom and attrition.

### Key Considerations
*   **Skills Matrix**: Matching task complexity with engineer seniority and domain knowledge.
*   **Capacity Planning**: Accounting for vacations, holidays, and maintenance work (KTLO).
*   **Context Switching**: Minimizing the number of active projects per engineer to preserve focus.
*   **Bus Factor**: Ensuring knowledge is distributed so the project survives if a key person leaves.

## Go Code Example
A simple scheduler that assigns tasks to engineers based on skill matching and available capacity.

```go
package main

import (
	"fmt"
	"sort"
)

type Skill string

const (
	GoLang    Skill = "Go"
	React     Skill = "React"
	Kubernetes Skill = "K8s"
)

type Engineer struct {
	Name     string
	Skills   map[Skill]bool
	Capacity int // Hours available per week
}

type Task struct {
	Name     string
	Required Skill
	Effort   int // Hours
}

type Scheduler struct {
	Engineers []*Engineer
}

func (s *Scheduler) Assign(tasks []Task) map[string]string {
	assignments := make(map[string]string) // Task -> Engineer

	// Sort tasks by effort (descending) to solve the "Bin Packing" problem efficiently
	sort.Slice(tasks, func(i, j int) bool {
		return tasks[i].Effort > tasks[j].Effort
	})

	for _, task := range tasks {
		assigned := false
		for _, eng := range s.Engineers {
			if eng.Skills[task.Required] && eng.Capacity >= task.Effort {
				eng.Capacity -= task.Effort
				assignments[task.Name] = eng.Name
				assigned = true
				break
			}
		}
		if !assigned {
			assignments[task.Name] = "UNASSIGNED (No Capacity/Skill)"
		}
	}
	return assignments
}

func main() {
	engineers := []*Engineer{
		{Name: "Alice", Skills: map[Skill]bool{GoLang: true, Kubernetes: true}, Capacity: 40},
		{Name: "Bob", Skills: map[Skill]bool{React: true}, Capacity: 20}, // Bob is part-time
	}

	tasks := []Task{
		{Name: "Backend API", Required: GoLang, Effort: 30},
		{Name: "Frontend Dashboard", Required: React, Effort: 15},
		{Name: "Cluster Setup", Required: Kubernetes, Effort: 10},
	}

	scheduler := Scheduler{Engineers: engineers}
	result := scheduler.Assign(tasks)

	for task, eng := range result {
		fmt.Printf("Task: %s -> %s\n", task, eng)
	}
}
```

## Interview Questions
**Q: How do you handle a situation where your best engineer is the bottleneck for everything?**
**A:** This is a "Bus Factor" of 1. I would stop assigning them critical path work immediately and instead assign them to mentor others, do code reviews, and document their knowledge. I would pair them with a junior engineer to transfer knowledge (pair programming) until the dependency is broken.

**Q: What do you do when you are understaffed for a critical deadline?**
**A:** I present the "Iron Triangle" to stakeholders: either we cut scope (deliver less), move the deadline (deliver later), or (less effectively) add resources/budget. I avoid simply "working harder" (crunch mode) as it creates technical debt and burnout.

**Q: How do you allocate resources for Technical Debt?**
**A:** I advocate for a fixed capacity allocation (e.g., 20% rule) where 20% of every sprint is dedicated to refactoring, tooling, or debt reduction, regardless of feature pressure.
