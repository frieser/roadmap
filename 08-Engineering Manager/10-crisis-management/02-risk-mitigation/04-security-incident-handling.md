## Summary
Security Incident Handling is a specialized form of incident response focused on breaches, vulnerabilities, and unauthorized access. It emphasizes containment, evidence preservation (forensics), and secure communication (so the attacker doesn't know you know).

## Detailed Explanation
Security incidents require a "Need to Know" basis. Panic can alert the attacker to wipe tracks.

### The SANS 6 Steps
1.  **Preparation**: Tools, playbooks, team.
2.  **Identification**: Detecting the breach.
3.  **Containment**: Stop the bleeding (isolate the host).
4.  **Eradication**: Remove the malware/access.
5.  **Recovery**: Restore systems.
6.  **Lessons Learned**: Post-mortem.

### Key Differences from Ops Incidents
*   **Adversarial**: There is an intelligent actor fighting back.
*   **Legal**: Data breach laws (GDPR/CCPA) impose strict notification deadlines.

## Go Code Example
Modeling a `SecurityEvent` handler that isolates a compromised "node" (struct) automatically.

```go
package main

import "fmt"

type Node struct {
	IPAddress string
	IsIsolated bool
	Status    string
}

type SecurityPolicy struct {
	AutoIsolateHighSev bool
}

func (p SecurityPolicy) HandleEvent(node *Node, severity string) {
	fmt.Printf("Handling %s event for %s\n", severity, node.IPAddress)
	
	if severity == "HIGH" || severity == "CRITICAL" {
		if p.AutoIsolateHighSev {
			p.Isolate(node)
		}
	}
}

func (p SecurityPolicy) Isolate(n *Node) {
	n.IsIsolated = true
	n.Status = "QUARANTINE"
	fmt.Printf("🛑 Node %s has been ISOLATED from the network.\n", n.IPAddress)
}

func main() {
	policy := SecurityPolicy{AutoIsolateHighSev: true}
	
	webServer := &Node{IPAddress: "192.168.1.50", Status: "ACTIVE"}
	
	// Simulation: Intrusion Detection System triggers
	policy.HandleEvent(webServer, "CRITICAL")
	
	fmt.Printf("Final Server Status: %s\n", webServer.Status)
}
```

## Interview Questions
**Q: When do you decide to shut down the system vs. keep it running to monitor the attacker?**
**A:** It's a trade-off between business impact and forensic value. If customer data is actively exfiltrating, we shut down immediately (Containment). If it's a low-level probe, we might observe briefly to understand the attack vector, but containment is usually priority #1.

**Q: Who triggers the data breach notification?**
**A:** That is a legal/executive decision, not engineering. My job is to provide the accurate scope (what data was touched?) to Legal so they can decide if we legally need to notify users.
