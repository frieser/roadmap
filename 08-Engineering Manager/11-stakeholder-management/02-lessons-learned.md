## Summary
"Lessons Learned" is the systematic capture of insights from projects, sprints, and incidents. Unlike a post-mortem (which is specific to an incident), lessons learned cover the entire project lifecycle—what went right, what went wrong, and what we would do differently.

## Detailed Explanation
It transforms "experience" into "wisdom."

### The Retrospective
The primary vehicle for capturing lessons is the Sprint Retrospective or Project Post-Implementation Review (PIR).
*   **Start/Stop/Continue**: Simple framework.
*   **Sailboat**: Anchors (drag), Wind (drivers), Rocks (risks).

### Closing the Loop
Lessons are useless if filed away. They must update the *process*. If we learned "estimation was off because of unknown APIs," the new process is "Prototype APIs before estimation."

## Go Code Example
Modeling a `RetrospectiveBoard` that aggregates feedback and highlights top action items.

```go
package main

import (
	"fmt"
	"sort"
)

type FeedbackType string

const (
	WentWell      FeedbackType = "Went Well"
	ToImprove     FeedbackType = "To Improve"
	ActionItem    FeedbackType = "Action Item"
)

type Feedback struct {
	Type  FeedbackType
	Text  string
	Votes int
}

func GetTopItems(items []Feedback) []Feedback {
	sort.Slice(items, func(i, j int) bool {
		return items[i].Votes > items[j].Votes
	})
	return items
}

func main() {
	board := []Feedback{
		{Type: WentWell, Text: "Deployed on time", Votes: 5},
		{Type: ToImprove, Text: "QA environment was flaky", Votes: 12},
		{Type: ToImprove, Text: "Requirements changed late", Votes: 8},
	}

	fmt.Println("Top Retro Items:")
	for _, item := range GetTopItems(board) {
		fmt.Printf("[%s] %s (%d votes)\n", item.Type, item.Text, item.Votes)
	}
}
```

## Interview Questions
**Q: How do you run a retrospective when the team is morale is low?**
**A:** I focus on "small wins" and psychological safety. I might run a "Vent Session" first to clear the air, acknowledging the pain, then pivot to "What is the *one* thing within our control that we can fix next week?"

**Q: Give an example of a lesson learned that changed your management style.**
**A:** I learned that "protecting the team" from all external noise resulted in them missing business context. I now share the "why" and the business pressure, but filter the *urgency* and panic.
