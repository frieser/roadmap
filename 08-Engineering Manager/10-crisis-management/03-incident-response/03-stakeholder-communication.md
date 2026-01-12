## Summary
Stakeholder Communication is vital during a crisis. Silence breeds distrust. The goal is to provide regular, accurate, and calibrated updates to different audiences (Customers, Executives, Support Team) without distracting the engineers fixing the problem.

## Detailed Explanation
Different stakeholders need different information.
*   **Customers**: "We are aware of an issue. We are working on it." (Reassurance)
*   **Executives**: "Impact is X. Risk is Y. ETA is Z." (Business Impact)
*   **Support**: "Tell customers X. Do not promise Y." (Scripting)

### The Cadence
Update every 30 minutes (or agreed interval), *even if there is no news*. "We are still investigating" is better than silence.

## Go Code Example
Modeling a `Notifier` interface with varying implementations for different stakeholder channels.

```go
package main

import "fmt"

type Message struct {
	InternalDetail string
	PublicSafe     string
}

type Notifier interface {
	Notify(msg Message)
}

type SlackNotifier struct {
	Channel string
}

func (s SlackNotifier) Notify(msg Message) {
	fmt.Printf("[Slack %s] %s\n", s.Channel, msg.InternalDetail)
}

type EmailNotifier struct {
	List string
}

func (e EmailNotifier) Notify(msg Message) {
	fmt.Printf("[Email to %s] %s\n", e.List, msg.PublicSafe)
}

type StatusPageNotifier struct {}

func (sp StatusPageNotifier) Notify(msg Message) {
	fmt.Printf("[StatusPage.io] Update: %s\n", msg.PublicSafe)
}

func main() {
	// Incident update
	update := Message{
		InternalDetail: "DB Master failed, promoting replica. Data corruption possible.",
		PublicSafe:     "We are experiencing database connectivity issues. Engineering is mitigating.",
	}

	notifiers := []Notifier{
		SlackNotifier{Channel: "#execs"},
		EmailNotifier{List: "customers@company.com"},
		StatusPageNotifier{},
	}

	for _, n := range notifiers {
		n.Notify(update)
	}
}
```

## Interview Questions
**Q: Why shouldn't we tell customers the technical details immediately?**
**A:** Accuracy matters. Early diagnosis is often wrong. Telling customers "It's a DNS issue" and then retracting it looks incompetent. Stick to symptoms ("Login is slow") until the root cause is confirmed.

**Q: How do you handle a breach of SLA with a major client during an outage?**
**A:** Transparency and proactive communication. Don't wait for them to ask. "We know we are impacting your business. Here is our plan." After the incident, we discuss credits/refunds, but during the fire, we focus on the fix.
