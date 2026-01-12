## Summary
Emotional Intelligence (EQ) is the ability to understand and manage your own emotions, and those of the people around you. For leaders, high EQ is often more critical than high IQ. It consists of self-awareness, self-regulation, motivation, empathy, and social skills. It enables managers to handle pressure, resolve conflict, and build strong teams.

## Detailed Explanation
### The 4 Quadrants of EQ
1.  **Self-Awareness**: Recognizing your own emotions and triggers. ("I am angry right now").
2.  **Self-Management**: Controlling impulsive feelings and behaviors. ("I will not send this Slack message while angry").
3.  **Social Awareness**: Empathy; understanding the emotions/needs of others. ("She is quiet today, something is wrong").
4.  **Relationship Management**: Developing and maintaining good relationships. (Inspiring, influencing, managing conflict).

### Amygdala Hijack
The immediate, overwhelming emotional response that bypasses the rational brain. High EQ involves recognizing this hijack and hitting the "pause" button.

### Go Code Example: The EQ Filter
This code models the process of self-regulation: taking an input event, checking emotional state, and filtering the response to ensure it is constructive.

```go
package leadership

import "fmt"

// EmotionalState represents the internal feeling.
type EmotionalState struct {
	Level    int    // 1-10
	Type     string // "Calm", "Angry", "Stressed"
}

// Event represents an external trigger.
type Event struct {
	Message string
	Sender  string
}

// Manager represents the leader processing the event.
type Manager struct {
	State EmotionalState
}

// ProcessResponse applies Self-Regulation
func (m *Manager) ProcessResponse(e Event) string {
	// 1. Self-Awareness: Recognize state
	if m.State.Type == "Angry" && m.State.Level > 7 {
		// 2. Self-Regulation: Pause logic
		return fmt.Sprintf("[Internal Log] Triggered by '%s'. State is Angry (%d). PAUSING response. Will reply later.", e.Message, m.State.Level)
	}

	// Default constructive response
	return fmt.Sprintf("Thanks %s, let's discuss '%s' to find a solution.", e.Sender, e.Message)
}

func main() {
	// Scenario: Production is down, Manager is stressed/angry
	mgr := Manager{
		State: EmotionalState{Level: 9, Type: "Angry"},
	}

	trigger := Event{
		Sender:  "Junior Dev",
		Message: "I accidentally deleted the production database.",
	}

	response := mgr.ProcessResponse(trigger)
	fmt.Println(response)
	
	// Outcome: Manager avoids yelling, prevents damage to relationship.
}
```

## Interview Questions
**Q: Describe a time you lost your temper at work. How did you handle it?**
**A:** (Use the STAR method). "During a critical launch, a dependency broke. I was exhausted and snapped at a QA engineer who asked a basic question. I immediately recognized my 'Amygdala Hijack' (Self-Awareness). I apologized publicly in the channel within 10 minutes (Relationship Management), explaining I was stressed and that their question was valid. I then took a 5-minute walk to reset (Self-Regulation)."

**Q: How do you assess the emotional state of your team remotely?**
**A:** Since I can't read body language as easily, I pay attention to changes in patterns. Are they quieter in Slack? are cameras off more often? Is the tone of their PR comments becoming shorter? I explicitly ask "How are you feeling about X?" rather than just "How is X going?" to open the door for emotional sharing (Social Awareness).

**Q: Why is empathy important in engineering management?**
**A:** Engineering is intellectual, creative work. People cannot code well if they are fearful or anxious. Empathy allows me to understand what is blocking them—whether it's technical debt or personal stress—and remove those blockers so they can perform. It builds the psychological safety required for innovation.
