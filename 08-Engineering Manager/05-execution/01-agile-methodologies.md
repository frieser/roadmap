## Summary
Agile methodologies are iterative approaches to software development that prioritize flexibility, customer collaboration, and rapid delivery of functional software. Unlike traditional Waterfall models, Agile focuses on breaking projects into small, manageable increments (sprints) to allow for continuous improvement and adaptation to changing requirements.

## Detailed Explanation
Agile is not a single process but a mindset defined by the Agile Manifesto. It values individuals and interactions over processes and tools, working software over comprehensive documentation, customer collaboration over contract negotiation, and responding to change over following a plan.

### Key Frameworks
*   **Scrum**: Structured approach with fixed-length sprints (usually 2 weeks), defined roles (Scrum Master, Product Owner, Team), and specific ceremonies (Daily Standup, Sprint Planning, Review, Retrospective).
*   **Kanban**: continuous flow method using a visual board to manage work in progress (WIP). It focuses on optimizing cycle time and flow efficiency.
*   **XP (Extreme Programming)**: Emphasizes technical practices like Test-Driven Development (TDD), Pair Programming, and Continuous Integration.

### Benefits
*   **Faster Feedback Loop**: Stakeholders see progress early and often.
*   **Risk Reduction**: incremental delivery isolates failures.
*   **Higher Quality**: Continuous testing and integration.

## Go Code Example
Modeling Agile methodologies using Go interfaces to demonstrate the difference between Scrum and Kanban workflows.

```go
package main

import (
	"fmt"
	"time"
)

// Methodology defines the core behavior of an Agile process
type Methodology interface {
	Plan()
	Execute()
	Review()
}

// Sprint representing a fixed time-box in Scrum
type Sprint struct {
	Number   int
	Duration time.Duration
	Goal     string
}

// Scrum implementation
type Scrum struct {
	TeamName string
	Sprints  []Sprint
}

func (s *Scrum) Plan() {
	fmt.Printf("[%s] Planning Sprint %d: Defining user stories and estimating points.\n", s.TeamName, len(s.Sprints)+1)
}

func (s *Scrum) Execute() {
	fmt.Printf("[%s] Sprint Execution: Daily standups and heads-down coding for %s.\n", s.TeamName, s.Sprints[len(s.Sprints)-1].Duration)
}

func (s *Scrum) Review() {
	fmt.Printf("[%s] Sprint Review: Demoing increment to stakeholders.\n", s.TeamName)
}

// Kanban implementation
type Kanban struct {
	BoardName   string
	WIPLimit    int
	CurrentTask string
}

func (k *Kanban) Plan() {
	fmt.Printf("[%s] Just-in-Time Planning: Pulling next highest priority task into 'To Do'.\n", k.BoardName)
}

func (k *Kanban) Execute() {
	if k.CurrentTask != "" {
		fmt.Printf("[%s] Flow Execution: Working on '%s' (WIP Limit: %d).\n", k.BoardName, k.CurrentTask, k.WIPLimit)
	}
}

func (k *Kanban) Review() {
	fmt.Printf("[%s] Flow Review: Analyzing Cycle Time and Lead Time metrics.\n", k.BoardName)
}

func main() {
	// Scrum Simulation
	scrumTeam := &Scrum{TeamName: "Alpha Squad", Sprints: []Sprint{{Number: 1, Duration: 2 * 7 * 24 * time.Hour, Goal: "MVP"}}}
	runCycle(scrumTeam)

	// Kanban Simulation
	kanbanTeam := &Kanban{BoardName: "Ops Flow", WIPLimit: 3, CurrentTask: "Fix Prod Incident"}
	runCycle(kanbanTeam)
}

func runCycle(m Methodology) {
	m.Plan()
	m.Execute()
	m.Review()
	fmt.Println("--- Cycle Complete ---")
}
```

## Interview Questions
**Q: What is the main difference between Scrum and Kanban?**
**A:** Scrum is based on fixed time-boxed iterations (sprints) with defined roles and ceremonies, emphasizing predictability within that box. Kanban is a flow-based method that focuses on continuous delivery and limiting Work In Progress (WIP) to optimize throughput, without fixed iterations.

**Q: How do you handle changing requirements in the middle of a Sprint?**
**A:** Generally, scope changes within a sprint are discouraged to protect the team's focus. If critical, the Product Owner and team negotiate: either a lower-priority item is swapped out to maintain total effort, or if the change drastically invalidates the sprint goal, the sprint may be abnormally terminated and replanned.

**Q: Explain the role of the Product Owner vs. the Scrum Master.**
**A:** The Product Owner is responsible for *what* is built (vision, backlog prioritization, ROI). The Scrum Master is responsible for the *process* (coaching the team, removing impediments, facilitating ceremonies) and acts as a servant-leader.
