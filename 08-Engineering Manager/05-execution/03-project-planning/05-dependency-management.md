## Summary
Dependency Management in project planning involves identifying, visualizing, and organizing tasks that rely on the completion of other tasks. Dependencies dictate the sequence of work and determine the project's critical path. Poor dependency management leads to "blocked" developers and schedule slippage.

## Detailed Explanation
Dependencies can be internal (Task B needs Task A) or external (Team X needs API from Team Y).

### Types of Dependencies
*   **Finish-to-Start (FS)**: Task A must finish before B starts (Standard).
*   **Start-to-Start (SS)**: Task B cannot start until A starts.
*   **Finish-to-Finish (FF)**: Task B cannot finish until A finishes.
*   **Start-to-Finish (SF)**: Task B cannot finish until A starts (Rare).

### Management Strategies
*   **Decoupling**: Using interfaces, mocks, or feature flags to allow parallel work despite dependencies.
*   **Visualizing**: Using Gantt charts or Dependency Graphs (DAGs).
*   **Critical Path Method (CPM)**: Identifying the longest path of dependent tasks to determine the shortest possible project duration.

## Go Code Example
Using a Directed Acyclic Graph (DAG) to model task dependencies and performing a Topological Sort to determine the correct execution order.

```go
package main

import (
	"fmt"
)

type Graph struct {
	Vertices int
	Adj      map[string][]string
	InDegree map[string]int
}

func NewGraph() *Graph {
	return &Graph{
		Adj:      make(map[string][]string),
		InDegree: make(map[string]int),
	}
}

func (g *Graph) AddDependency(dependency, task string) {
	g.Adj[dependency] = append(g.Adj[dependency], task)
	g.InDegree[task]++
	if _, exists := g.InDegree[dependency]; !exists {
		g.InDegree[dependency] = 0
	}
}

// TopologicalSort returns a linear ordering of tasks
func (g *Graph) TopologicalSort() ([]string, error) {
	var queue []string
	var result []string

	// Initialize queue with tasks having 0 dependencies
	for task, degree := range g.InDegree {
		if degree == 0 {
			queue = append(queue, task)
		}
	}

	for len(queue) > 0 {
		u := queue[0]
		queue = queue[1:]
		result = append(result, u)

		for _, v := range g.Adj[u] {
			g.InDegree[v]--
			if g.InDegree[v] == 0 {
				queue = append(queue, v)
			}
		}
	}

	if len(result) != len(g.InDegree) {
		return nil, fmt.Errorf("circular dependency detected")
	}

	return result, nil
}

func main() {
	g := NewGraph()

	// "Frontend" depends on "API Design"
	// "Backend" depends on "API Design"
	// "Integration" depends on "Frontend" AND "Backend"
	
	g.AddDependency("API Design", "Frontend")
	g.AddDependency("API Design", "Backend")
	g.AddDependency("Frontend", "Integration")
	g.AddDependency("Backend", "Integration")
	g.AddDependency("DB Setup", "Backend") // DB Setup must happen before Backend starts? No, other way.
	
	// Correction: "DB Setup" is a dependency FOR Backend.
	// So: AddDependency("DB Setup", "Backend") means DB Setup -> Backend
	
	fmt.Println("Resolving Build Order...")
	order, err := g.TopologicalSort()
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		for i, task := range order {
			fmt.Printf("%d. %s\n", i+1, task)
		}
	}
}
```

## Interview Questions
**Q: How do you handle a circular dependency between two teams?**
**A:** Circular dependencies (Team A needs B, B needs A) are architectural smells. I resolve them by:
1.  Defining a shared Interface/Contract that both can build against (Dependency Inversion).
2.  Creating a third "Common" module that both depend on.
3.  Using Mocks/Stubs to allow development to proceed in parallel before integration.

**Q: What is the 'Critical Path'?**
**A:** It is the sequence of tasks that determines the minimum time needed for an operation. If any task on the critical path is delayed, the entire project is delayed. Non-critical tasks have "float" or "slack" time.

**Q: How do you track external dependencies?**
**A:** I maintain a specific "Dependency Log" or board column. I establish regular syncs with the external owners and treat the dependency delivery date as a Milestone in my own project plan, adding a buffer for safety.
