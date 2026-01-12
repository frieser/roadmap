# Architectural Decision Making

## Summary
Architectural Decision Making is the process of selecting technical approaches to solve significant problems. It requires balancing trade-offs (e.g., consistency vs. availability, speed vs. maintainability) and documenting the "why" behind decisions. The industry standard for documentation is the Architecture Decision Record (ADR).

## Detailed Explanation
Architecture is often described as "the decisions that are hard to change later." Therefore, these decisions require a structured approach, not just "gut feeling."

### The Process
1.  **Define the Problem**: Clearly state what needs to be solved.
2.  **Identify Constraints**: Budget, timeline, team skills, existing infrastructure.
3.  **Explore Options**: Generate multiple viable solutions (e.g., SQL vs. NoSQL, Monolith vs. Microservices).
4.  **Evaluate Trade-offs**: Analyze the pros and cons of each option.
5.  **Decide**: Select the option that best fits the context (not necessarily the "best" tech).
6.  **Document**: Write an ADR.

### Architecture Decision Records (ADR)
An ADR is a lightweight text file that captures a decision. It typically includes:
- **Title**: Short and descriptive.
- **Status**: Proposed, Accepted, Deprecated.
- **Context**: The forces at play.
- **Decision**: What we are doing.
- **Consequences**: The positive and negative outcomes (accepted tech debt).

### Decision Making Models
- **Consensus**: Everyone agrees (slow, high buy-in).
- **Consent**: "Good enough for now, safe enough to try" (faster).
- **Consultative**: Leader decides after hearing input (fast, clear accountability).

## Go Code Example
This example models a Decision Matrix, a tool often used to quantify architectural choices based on weighted criteria.

```go
package main

import (
	"fmt"
)

// Criterion represents a factor in the decision (e.g., "Performance")
type Criterion struct {
	Name   string
	Weight float64 // Importance (1-10)
}

// Option represents a solution choice (e.g., "PostgreSQL")
type Option struct {
	Name   string
	Scores map[string]float64 // Score per criterion (1-10)
}

// DecisionMatrix evaluates options
type DecisionMatrix struct {
	Criteria []Criterion
	Options  []Option
}

func (dm DecisionMatrix) Evaluate() {
	fmt.Println("--- Architectural Decision Matrix Results ---")
	
	bestScore := -1.0
	bestOption := ""

	for _, opt := range dm.Options {
		totalScore := 0.0
		for _, crit := range dm.Criteria {
			score := opt.Scores[crit.Name]
			weighted := score * crit.Weight
			totalScore += weighted
		}
		fmt.Printf("Option: %s | Total Score: %.2f\n", opt.Name, totalScore)
		
		if totalScore > bestScore {
			bestScore = totalScore
			bestOption = opt.Name
		}
	}
	
	fmt.Printf("\n🏆 Recommended Decision: %s\n", bestOption)
}

func main() {
	// Scenario: Choosing a Message Broker
	matrix := DecisionMatrix{
		Criteria: []Criterion{
			{Name: "Throughput", Weight: 9.0},
			{Name: "Ease of Use", Weight: 4.0},
			{Name: "Team Familiarity", Weight: 7.0},
		},
		Options: []Option{
			{
				Name: "Kafka",
				Scores: map[string]float64{
					"Throughput": 10.0,
					"Ease of Use": 3.0,
					"Team Familiarity": 5.0,
				},
			},
			{
				Name: "RabbitMQ",
				Scores: map[string]float64{
					"Throughput": 7.0,
					"Ease of Use": 8.0,
					"Team Familiarity": 9.0, // Team knows it well
				},
			},
		},
	}

	matrix.Evaluate()
}
```

## Interview Questions
1.  **What is an ADR and why would you use one?**
    *   *Focus*: Institutional memory, preventing circular debates, documenting the context of past decisions.
2.  **Tell me about a difficult architectural decision you had to make. How did you decide?**
    *   *Focus*: Structured thinking, evaluating trade-offs, involving the team, handling disagreement.
3.  **How do you handle "Analysis Paralysis" in your team?**
    *   *Focus*: Time-boxing research, defining "good enough," bias for action, reversibility of decisions (Type 1 vs Type 2 decisions).
4.  **When would you choose a monolithic architecture over microservices today?**
    *   *Focus*: Complexity, team size, domain clarity, deployment overhead.
