## Summary
Emergency Protocols act as the "break glass in case of emergency" instructions. They define the immediate steps to take when a high-severity incident is detected, ensuring a standardized, calm, and effective initial reaction. They answer: "Who do I call?" and "What do I do first?"

## Detailed Explanation
Chaos defines the first 10 minutes of an incident. Protocols impose order.

### Protocol Steps
1.  **Acknowledge**: "I am looking at this."
2.  **Assess**: Is this SEV1 (Customer facing) or SEV3 (Internal annoyance)?
3.  **Escalate**: Page the Incident Commander (IC).
4.  **Contain**: Do not fix yet; stop the spread (e.g., rollback, feature flag off).
5.  **Communicate**: Create the Slack channel/Zoom bridge.

### Roles
*   **Incident Commander (IC)**: Runs the process, does NOT touch code.
*   **Scribe**: Writes down everything happening.
*   **Ops Lead**: Directs the technical fix.
*   **Comms Lead**: Updates stakeholders.

## Go Code Example
Modeling an `IncidentDispatcher` that assigns roles and spins up communication channels based on severity.

```go
package main

import "fmt"

type IncidentContext struct {
	ID       string
	Severity int
	Channels []string
	Roles    map[string]string
}

func InitializeIncident(id string, severity int) IncidentContext {
	ctx := IncidentContext{
		ID:       id,
		Severity: severity,
		Roles:    make(map[string]string),
	}

	// Protocol Logic
	if severity == 1 {
		fmt.Println("🚨 SEV1 DETECTED. ACTIVATING FULL PROTOCOL.")
		ctx.Channels = append(ctx.Channels, "#incident-"+id, "#exec-updates")
		ctx.Roles["IC"] = "PagerDuty-OnCall-Manager"
		ctx.Roles["Comms"] = "PagerDuty-Marketing"
	} else {
		fmt.Println("⚠️ Minor Incident. Standard protocol.")
		ctx.Channels = append(ctx.Channels, "#incident-"+id)
		ctx.Roles["IC"] = "TechLead"
	}
	
	return ctx
}

func main() {
	incident := InitializeIncident("2024-500", 1)
	fmt.Printf("Assigned IC: %s\n", incident.Roles["IC"])
	fmt.Printf("Created Channels: %v\n", incident.Channels)
}
```

## Interview Questions
**Q: Why should the Incident Commander NOT touch the code?**
**A:** The IC needs a holistic view of the situation. If they are heads-down debugging a database query, they lose situational awareness, miss stakeholder updates, and can't coordinate parallel work streams.

**Q: What is the first thing you do when you get a SEV1 page?**
**A:** Acknowledge the page so the escalation policy stops ringing others. Then, log into the incident channel and state "I am checking in."
