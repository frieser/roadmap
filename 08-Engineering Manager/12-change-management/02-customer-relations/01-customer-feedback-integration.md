# Customer Feedback Integration

## Summary
Customer Feedback Integration is the systematic process of collecting, analyzing, and acting on user input. For EMs, this means closing the loop between "What users say" and "What engineers build." It prevents the team from building perfect code for the wrong problem.

## Detailed Explanation

### 1. Channels of Feedback
*   **Quantitative**: Analytics (Amplitude, Google Analytics), Crash Logs (Sentry). "What are they doing?"
*   **Qualitative**: Support Tickets (Zendesk), NPS Surveys, User Interviews. "Why are they doing it?"
*   **Indirect**: Sales calls, Social Media (Twitter/Reddit).

### 2. The "Voice of the Customer" (VoC) Program
*   Establish a ritual where engineers read/hear direct feedback.
*   **Example**: "Support Ticket Roulette" (Engineers solve 1 ticket/week).
*   **Outcome**: Builds empathy. Fixing a bug feels different when you know it ruined "Bob from Accounting's" weekend.

### 3. Categorization (Triage)
*   **Bugs**: Fix immediately (if critical) or schedule.
*   **Feature Requests**: Add to "Idea Backlog" (not Development Backlog). Validate demand.
*   **UX Friction**: Hard to quantify but causes churn. Requires "Design Debt" cleanup.

## Go Code Example: Feedback Sentiment Analyzer
This conceptual example sorts feedback into categories based on keywords to help prioritization.

```go
package main

import (
	"fmt"
	"strings"
)

type Feedback struct {
	User    string
	Message string
	Source  string
}

type Category string

const (
	Bug      Category = "BUG"
	Feature  Category = "FEATURE"
	Praise   Category = "PRAISE"
	ChrunRisk Category = "CHURN_RISK"
)

func Categorize(f Feedback) Category {
	msg := strings.ToLower(f.Message)
	
	if strings.Contains(msg, "cancel") || strings.Contains(msg, "expensive") || strings.Contains(msg, "competitor") {
		return ChrunRisk
	}
	if strings.Contains(msg, "error") || strings.Contains(msg, "crash") || strings.Contains(msg, "bug") {
		return Bug
	}
	if strings.Contains(msg, "wish") || strings.Contains(msg, "add") || strings.Contains(msg, "missing") {
		return Feature
	}
	if strings.Contains(msg, "love") || strings.Contains(msg, "great") {
		return Praise
	}
	return "UNKNOWN"
}

func main() {
	inbox := []Feedback{
		{"user1", "The app crashes when I upload a PDF.", "Support"},
		{"user2", "I wish there was a dark mode.", "Twitter"},
		{"user3", "I am cancelling my subscription, too expensive.", "Email"},
		{"user4", "Great job on the new update!", "App Store"},
	}

	fmt.Println("--- Feedback Triage ---")
	for _, f := range inbox {
		cat := Categorize(f)
		priority := "Low"
		if cat == Bug || cat == ChrunRisk {
			priority = "HIGH"
		}
		
		fmt.Printf("[%s] %s (Priority: %s)\n", cat, f.Message, priority)
	}
}
```

## Interview Questions

### Q: "How do you handle a feature request that one big customer demands but nobody else wants?"
**A:**
*   **The "Consultancy Trap"**: If we build custom code for one client, we become a dev shop, not a product company.
*   **Strategy**:
    *   Find the *root problem*. Is there a generic solution that helps everyone?
    *   If not, use Feature Flags to toggle it only for them (and maybe charge them for it).
    *   Or, politely say No. "It doesn't fit our roadmap."

### Q: "How do you connect engineers with customers?"
**A:**
*   **Ride-alongs**: Engineers sit on Sales/Support calls (mute).
*   **FullStory**: Watch session replays of users struggling with the UI. It's painful but effective.

### Q: "What is the difference between what customers *say* and what they *do*?"
**A:**
*   **Say**: "I want a dashboard with 50 widgets."
*   **Do**: They only ever check 1 number.
*   *Lesson*: Trust behavior (Analytics) over opinions (Surveys). Build for what they do.
