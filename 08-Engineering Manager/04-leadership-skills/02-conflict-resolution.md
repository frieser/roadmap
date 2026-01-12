## Summary
Conflict resolution is the process of resolving a dispute or a conflict by meeting at least some of each side's needs and addressing their interests. For engineering managers, this often involves mediating between engineers with differing technical opinions or resolving interpersonal friction. The goal is constructive resolution that strengthens the relationship and leads to better technical outcomes.

## Detailed Explanation
### Conflict Styles (Thomas-Kilmann)
1.  **Collaborating**: High assertiveness, high cooperation. "Win-win."
2.  **Compromising**: Moderate assertiveness/cooperation. "Split the difference."
3.  **Accommodating**: Low assertiveness, high cooperation. "You win."
4.  **Competing**: High assertiveness, low cooperation. "I win."
5.  **Avoiding**: Low assertiveness, low cooperation. (Rarely useful).

### The Resolution Process
1.  **De-escalate**: Move the conversation from public (Slack/PR) to private (Video/F2F).
2.  **Listen**: Let each side state their case without interruption.
3.  **Identify the Core Issue**: Is it technical? Personal? Resourcing?
4.  **Agree on Criteria**: "What are we optimizing for? Latency or Dev Speed?"
5.  **Commit**: Agree on a path forward.

### Go Code Example: The Mediator Pattern
This code models a conflict resolution system where a Mediator facilitates communication between two conflicting parties (Colleagues).

```go
package leadership

import "fmt"

// Conflict represents the issue at hand.
type Conflict struct {
	Topic     string
	Intensity int // 1-10
	IsSolved  bool
}

// Colleague represents an employee involved in the conflict.
type Colleague struct {
	Name     string
	Position string // e.g., "Use Postgres", "Use Mongo"
}

// Mediator is the interface for resolving disputes.
type Mediator interface {
	Resolve(c1, c2 Colleague, issue Conflict) Conflict
}

// EngineeringManager implements Mediator.
type EngineeringManager struct {
	Name string
}

func (em *EngineeringManager) Resolve(c1, c2 Colleague, issue Conflict) Conflict {
	fmt.Printf("Mediator %s: Bringing %s and %s together regarding '%s'.\n", 
		em.Name, c1.Name, c2.Name, issue.Topic)

	// Step 1: Identification
	fmt.Printf("- %s wants: %s\n", c1.Name, c1.Position)
	fmt.Printf("- %s wants: %s\n", c2.Name, c2.Position)

	// Step 2: Finding Common Ground (Simplified Logic)
	// In real life, this is the hard conversation.
	fmt.Println("Mediator: Let's focus on the business requirement: Stability.")

	issue.IsSolved = true
	fmt.Println("Result: Consensus reached. Conflict resolved.")
	
	return issue
}

func main() {
	em := &EngineeringManager{Name: "Sarah"}
	
	devA := Colleague{Name: "John", Position: "Refactor everything now"}
	devB := Colleague{Name: "Doe", Position: "Wait for feature freeze"}
	
	conflict := Conflict{Topic: "Tech Debt Strategy", Intensity: 8, IsSolved: false}
	
	em.Resolve(devA, devB, conflict)
}
```

## Interview Questions
**Q: How do you handle two senior engineers arguing over an architectural decision?**
**A:** I first ensure the debate is about the problem, not the people. I ask them to write down their arguments (RFC or Design Doc) focusing on trade-offs (Pros/Cons) rather than opinions. If they still can't agree, I intervene as the tie-breaker, making a decision based on business constraints (time, cost) so the team can commit and move forward.

**Q: Tell me about a time you had to resolve a personal conflict between team members.**
**A:** I had two developers who simply didn't get along, leading to passive-aggressive PR comments. I held separate 1:1s to understand the root cause (communication style differences). Then I facilitated a joint session where they used "I" statements ("I feel ignored when...") to express their frustrations. We established a "Team Charter" for communication which resolved the friction.

**Q: What is your default conflict resolution style?**
**A:** I default to **Collaborating**. I believe that in engineering, "conflict" often points to a constraint or a missing piece of data. By digging deep together, we often find a third option that is better than either of the original opposing views.
