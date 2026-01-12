## Summary
Team meetings (Standups, Planning, Retrospectives, All-Hands) are expensive activities that pull everyone away from deep work. Their purpose is alignment, decision making, and bonding. Effective team meetings must have a clear purpose, a defined agenda, and be rigorously time-boxed. If a meeting can be an email or a Slack thread, it should be.

## Detailed Explanation
### Common Meeting Types
*   **Standup**: 15 mins max. Unblock, synchronize. Not for problem-solving.
*   **Retrospective**: Continuous improvement. Safe space to discuss what went wrong/right.
*   **Sprint Planning**: Defining the "What" and "Why" for the next cycle.
*   **Demo**: celebrating progress and sharing knowledge.

### Best Practices
*   **No Agenda, No Attendance**: Decline meetings without a clear goal.
*   **Facilitator**: Rotate the role so everyone learns to run meetings.
*   **Action Items**: Leave with clear owners and due dates.

### Go Code Example: The Meeting Cost Calculator
This code calculates the financial cost of a meeting to help a manager decide if it's worth scheduling.

```go
package communication

import "fmt"

type Attendee struct {
	Name       string
	HourlyRate float64
}

type Meeting struct {
	Topic     string
	Duration  float64 // hours
	Attendees []Attendee
}

// CalculateCost determines the burn rate of the meeting
func (m *Meeting) CalculateCost() float64 {
	totalCost := 0.0
	for _, p := range m.Attendees {
		totalCost += p.HourlyRate * m.Duration
	}
	return totalCost
}

// ShouldSchedule returns a recommendation
func (m *Meeting) ShouldSchedule(valueEstimate float64) string {
	cost := m.CalculateCost()
	fmt.Printf("Meeting '%s' Cost: $%.2f\n", m.Topic, cost)
	
	if cost > valueEstimate {
		return "Recommendation: CANCEL. Send an email instead."
	}
	return "Recommendation: PROCEED. Value exceeds cost."
}

func main() {
	team := []Attendee{
		{"Dev 1", 100}, {"Dev 2", 100}, {"Dev 3", 100},
		{"Manager", 150}, {"Product Owner", 130},
	}

	// 1 hour sync with 5 people
	weeklySync := Meeting{
		Topic:     "Weekly Status Sync",
		Duration:  1.0,
		Attendees: team,
	}

	// Is the value of hearing status updates worth ~$600?
	fmt.Println(weeklySync.ShouldSchedule(200)) 
}
```

## Interview Questions
**Q: How do you handle a meeting that is going off-track?**
**A:** I intervene politely but firmly. "This is a great discussion, but it seems to be deep-diving into a specific solution. Let's take this offline with just the relevant folks so we can get through the rest of the agenda." (The "Parking Lot" technique).

**Q: Your standups are taking 30 minutes. What do you do?**
**A:** I re-iterate the purpose: Blockers and Sync, not problem-solving. I enforce the "1 minute per person" rule. I might introduce a token (like a ball) to pass around. If people are rambling, I ask them to write their update before the meeting. If the team is too big, I split the standup.

**Q: How do you make retrospectives effective?**
**A:** I vary the format (Start/Stop/Continue, Sailboat, Mad/Sad/Glad) to prevent boredom. I ensure we focus on *actionable* improvements, not just venting. We pick 1-2 top items to fix in the next sprint, and I assign owners. We review the previous sprint's action items at the start to ensure follow-through.
