# Team Traditions and Rituals

## Summary
Traditions and Rituals are the recurring events that bind a team together and reinforce its identity. They provide rhythm, predictability, and a sense of belonging. Unlike "Meetings" (which are for work), "Rituals" have a symbolic or cultural value.

## Detailed Explanation

### 1. Types of Rituals
*   **Synchronization**: Standups, Planning (Work rhythm).
*   **Celebration**: Launch parties, Gong ringing, "Ship It" awards.
*   **Connection**: Team lunches, Coffee chats, Game nights.
*   **Learning**: Brown bags, Hackathons, Book clubs.

### 2. The Power of "Inside Jokes"
*   Shared context creates a "tribal" feeling.
*   *Warning*: Ensure inside jokes don't alienate new hires (cliques). Explain them during onboarding.

### 3. Remote Rituals
*   Remote teams need *more* deliberate rituals because serendipity is gone.
*   *Examples*: "Question of the Day" in Slack, "Fika" (virtual coffee), "Music League" (playlist sharing).

## Go Code Example: Ritual Scheduler
This example models a calendar of team rituals.

```go
package main

import (
	"fmt"
)

type RitualType string

const (
	Sync      RitualType = "Sync"
	Bonding   RitualType = "Bonding"
	Learning  RitualType = "Learning"
)

type Ritual struct {
	Name      string
	Type      RitualType
	Frequency string
	Value     string
}

func (r Ritual) Describe() {
	fmt.Printf("[%s] %s (%s): %s\n", r.Type, r.Name, r.Frequency, r.Value)
}

func main() {
	rituals := []Ritual{
		{"Daily Standup", Sync, "Daily", "Unblock work"},
		{"Fika / Coffee", Bonding, "Wednesday", "Social connection (No work talk)"},
		{"Demo Day", Learning, "Friday", "Celebrate wins & show progress"},
		{"Retro", Sync, "Bi-weekly", "Continuous improvement"},
		{"Hackathon", Learning, "Quarterly", "Innovation & Fun"},
	}

	fmt.Println("--- Team Ritual Calendar ---")
	for _, r := range rituals {
		r.Describe()
	}
}
```

## Interview Questions

### Q: "Tell me about a ritual you created for a team."
**A:**
*   **Example**: "Fail Cake."
*   **Context**: The team was afraid of breaking prod.
*   **Ritual**: Whenever we caused an incident, we bought a cake. We ate it while discussing the post-mortem.
*   **Outcome**: It destigmatized failure and turned a painful moment into a bonding/learning moment.

### Q: "How do you handle a team that thinks rituals are a waste of time?"
**A:**
*   **Audit**: Are they right? Is the "Fun Friday" forced fun?
*   **Opt-in**: Make social events optional.
*   **Purpose**: Explain the *why*. "The retro isn't just a meeting; it's the only time we fix our process."
*   **Experiment**: "Let's try cancelling X for a month. If we miss it, we bring it back."

### Q: "What is the most important ritual for an engineering team?"
**A:**
*   **The Retrospective**. It is the engine of improvement. Without it, the team repeats the same mistakes forever.
