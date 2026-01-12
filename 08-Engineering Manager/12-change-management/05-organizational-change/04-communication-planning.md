# Communication Planning

## Summary
Communication Planning is the structured approach to disseminating information. In change management, *under-communication* is the root cause of anxiety and resistance. A plan ensures the right message reaches the right people at the right time via the right channel.

## Detailed Explanation

### 1. The Rule of 7
*   Marketing concept: Someone needs to hear a message 7 times before they internalize it.
*   Don't say it once in an email and assume it's done.
*   **Channels**: Email -> Slack -> All Hands -> 1:1s -> Docs -> Posters.

### 2. The Narrative Arc
1.  **The Context**: "The market is changing."
2.  **The Problem**: "Our old tech stack is too slow."
3.  **The Solution**: "We are moving to X."
4.  **The Ask**: "We need you to learn X."
5.  **The Future**: "We will be able to ship daily."

### 3. Timing
*   **Pre-announcement**: Influencing key opinion leaders (1:1).
*   **Announcement**: Synchronous (All Hands) to control the narrative.
*   **Follow-up**: Async Q&A (AMA) to handle digestion details.

## Go Code Example: Comms Checklist Generator
Generates a schedule of communications for a rollout.

```go
package main

import (
	"fmt"
)

type Event struct {
	Day     int
	Channel string
	Audience string
	Message string
}

func GeneratePlan(launchDay int) []Event {
	return []Event{
		{launchDay - 14, "1:1s", "Tech Leads", "Pre-wire: Get feedback & buy-in."},
		{launchDay - 7,  "Email", "All Eng", "Teaser: Something is coming regarding CI/CD."},
		{launchDay,      "All Hands", "All Company", "Launch: Announce the new system live."},
		{launchDay,      "Slack", "All Eng", "Link to Docs & Migration Guide."},
		{launchDay + 1,  "Office Hours", "Users", "Open Q&A help session."},
		{launchDay + 30, "Email", "All Eng", "Success Metrics: How it's going."},
	}
}

func main() {
	plan := GeneratePlan(0)
	
	fmt.Println("--- Communication Plan ---")
	for _, e := range plan {
		fmt.Printf("T%+d [%s -> %s]: %s\n", e.Day, e.Channel, e.Audience, e.Message)
	}
}
```

## Interview Questions

### Q: "How do you ensure your message was understood?"
**A:**
*   **Feedback Loop**: Ask "What are your concerns?" rather than "Any questions?"
*   **Repeat Back**: Ask a manager, "How are you going to explain this to your team?"
*   **Pulse Survey**: Anonymous check-in. "Do you understand *why* we are changing?"

### Q: "Why announce bad news (layoffs/cancellations) synchronously?"
**A:**
*   **Respect**: People deserve to see your face.
*   **Control**: Prevents rumors from exploding in Slack before you finish the sentence.
*   **Empathy**: Allows you to read the room and adjust tone.

### Q: "How do you handle leaks?"
**A:**
*   **Acknowledge**: "Yes, you may have heard rumors."
*   **Accelerate**: If a leak happens, move the announcement up. Uncertainty is worse than bad news.
