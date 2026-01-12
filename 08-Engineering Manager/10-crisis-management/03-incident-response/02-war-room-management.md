## Summary
War Room Management is the art of coordinating a group of engineers in real-time during a crisis. It involves maintaining a single source of truth (the bridge/call), ensuring clear communication discipline, and preventing "too many cooks in the kitchen."

## Detailed Explanation
A War Room (or Incident Bridge) can easily become chaotic with 20 people shouting suggestions.

### Rules of Engagement
1.  **One Voice**: Only the IC directs traffic.
2.  **Clear Handoffs**: "Alice, you are investigating the DB. Bob, you are checking the Load Balancer. Report back in 5 minutes."
3.  **Silence is Golden**: Keep the channel clear for status updates. Speculation goes in a side thread.
4.  **No Tourists**: Executives or curious engineers should look at the status page/channel, not join the voice call and ask "What's the status?"

## Go Code Example
Modeling a `WarRoom` structure that manages active participants and limits noise.

```go
package main

import "fmt"

type Participant struct {
	Name string
	Role string // "IC", "Investigator", "Observer"
}

type WarRoom struct {
	ActiveParticipants []Participant
	MutedObservers     []Participant
}

func (w *WarRoom) Join(p Participant) {
	if p.Role == "Observer" {
		w.MutedObservers = append(w.MutedObservers, p)
		fmt.Printf("%s joined as Observer (Muted)\n", p.Name)
	} else {
		w.ActiveParticipants = append(w.ActiveParticipants, p)
		fmt.Printf("%s joined as %s (Active)\n", p.Name, p.Role)
	}
}

func (w *WarRoom) Broadcast(msg string, sender Participant) {
	if sender.Role == "Observer" {
		fmt.Println("Error: Observers cannot broadcast to the War Room.")
		return
	}
	fmt.Printf("[WAR ROOM] %s: %s\n", sender.Name, msg)
}

func main() {
	room := &WarRoom{}
	ic := Participant{Name: "Sarah", Role: "IC"}
	dev := Participant{Name: "Mike", Role: "Investigator"}
	ceo := Participant{Name: "Elon", Role: "Observer"}

	room.Join(ic)
	room.Join(dev)
	room.Join(ceo)

	room.Broadcast("Status update?", ceo) // Should fail
	room.Broadcast("Rolling back latest deploy.", dev) // Should succeed
}
```

## Interview Questions
**Q: How do you handle an executive who joins the war room and demands an ETA?**
**A:** The IC should intercept: "We are currently diagnosing the issue. An ETA is speculative right now. Please monitor the #exec-updates channel; we will post a confirmed status there in 15 minutes."

**Q: What do you do if the team is stuck and panic is setting in?**
**A:** I call a "Time Out." I ask everyone to stop typing/talking for 30 seconds, take a breath, and recap exactly what we know (facts) vs. what we guess (hypotheses). Then we pick ONE hypothesis to validate.
