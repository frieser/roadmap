## Summary
Project Management involves the application of processes, methods, skills, knowledge, and experience to achieve specific project objectives according to the project acceptance criteria within agreed parameters. It balances the "Iron Triangle" of constraints: Scope, Time, and Cost, while maintaining Quality.

## Detailed Explanation
Project Management provides the framework for executing work. It bridges the gap between high-level strategy and day-to-day execution.

### Key Phases (PMI Model)
1.  **Initiation**: Defining the project at a broad level (Project Charter).
2.  **Planning**: Developing a roadmap (Gantt charts, resource allocation, risk plans).
3.  **Execution**: The team does the actual work.
4.  **Monitoring & Controlling**: Tracking progress against the plan and adjusting as needed.
5.  **Closing**: Formal termination, handovers, and post-mortem.

### Methodologies
*   **Waterfall**: Linear, sequential phases. Good for construction or hardware with fixed requirements.
*   **Agile**: Iterative and incremental. Best for software with evolving requirements.
*   **Hybrid**: Mixing Waterfall planning with Agile execution.

## Go Code Example
Modeling the Project Management "Iron Triangle" (Time, Scope, Cost) and a Project Manager's state machine.

```go
package main

import (
	"fmt"
	"errors"
)

type ProjectStatus string

const (
	StatusInitiation ProjectStatus = "INITIATION"
	StatusPlanning   ProjectStatus = "PLANNING"
	StatusExecution  ProjectStatus = "EXECUTION"
	StatusClosing    ProjectStatus = "CLOSING"
)

// Constraint represents the Iron Triangle
type Constraint struct {
	Scope int // Story points
	Time  int // Days
	Cost  int // Budget units
}

type Project struct {
	Name       string
	Status     ProjectStatus
	Constraint Constraint
	Quality    float64 // 0.0 to 1.0
}

// AdjustScope demonstrates the trade-off principle: "Fast, Good, Cheap - Pick two"
func (p *Project) AdjustScope(newScope int) {
	fmt.Printf("Adjusting scope from %d to %d...\n", p.Constraint.Scope, newScope)
	diff := newScope - p.Constraint.Scope
	
	// Naive model: If scope increases, either time or cost must increase, or quality drops
	if diff > 0 {
		p.Constraint.Time += diff / 2
		p.Constraint.Cost += diff * 10
		p.Quality -= 0.05 // Adding scope late often hurts quality
		fmt.Println("Impact: Increased Time and Cost, slight Quality risk.")
	}
	p.Constraint.Scope = newScope
}

func (p *Project) NextPhase() error {
	switch p.Status {
	case StatusInitiation:
		p.Status = StatusPlanning
	case StatusPlanning:
		p.Status = StatusExecution
	case StatusExecution:
		p.Status = StatusClosing
	case StatusClosing:
		return errors.New("project already closed")
	}
	fmt.Printf("Transitioned to %s\n", p.Status)
	return nil
}

func main() {
	proj := Project{
		Name:   "Migration 2.0",
		Status: StatusInitiation,
		Constraint: Constraint{Scope: 100, Time: 30, Cost: 5000},
		Quality: 1.0,
	}

	proj.NextPhase() // To Planning
	proj.NextPhase() // To Execution
	
	// Scope creep happens during execution
	proj.AdjustScope(150)
	
	fmt.Printf("Final State: %+v\n", proj)
}
```

## Interview Questions
**Q: How do you handle a project that is falling behind schedule?**
**A:** First, diagnose the root cause (resource shortage, unclear requirements, technical blockers). Then, options include: reducing scope (cutting nice-to-haves), adding resources (though this has diminishing returns per Brooks' Law), or extending the timeline. I would communicate the delay and options to stakeholders early with a recommendation.

**Q: Explain the concept of the 'Iron Triangle'.**
**A:** The Iron Triangle refers to the three constraints of project management: Scope, Time, and Cost. You cannot change one without affecting at least one of the others. For example, if you increase Scope, you must either increase Time (deadline) or Cost (resources), otherwise Quality (often considered the fourth dimension) will suffer.

**Q: What is a critical path in project planning?**
**A:** The critical path is the longest sequence of dependent tasks in a project plan that must be completed on time for the project to complete by its deadline. Any delay on the critical path directly delays the project finish date.
