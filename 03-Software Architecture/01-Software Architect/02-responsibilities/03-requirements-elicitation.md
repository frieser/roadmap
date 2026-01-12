---
---

## Summary
Requirements Elicitation is the process of discovering the requirements for a system by communicating with customers, system users, and others who have a stake in the system development. For an architect, the primary focus is not just functional requirements ("What does it do?"), but critically, the **Non-Functional Requirements (NFRs)** or Quality Attributes ("How does it behave?").

## Detailed Explanation

A Software Architect bridges the gap between vague business desires and concrete technical specifications.

### Key Responsibilities
1.  **Stakeholder Analysis**: Identifying who cares about the system (Business, Security, Operations, Users) and what they value.
2.  **Eliciting NFRs**: Using specific techniques to quantify abstract goals.
    *   *Bad NFR*: "The system must be fast."
    *   *Good NFR*: "The system must process 10,000 transactions per second with 99th percentile latency under 200ms."
3.  **Trade-off Analysis**: Understanding that NFRs often conflict (e.g., Security vs. Usability, Consistency vs. Availability) and negotiating the right balance.

### Techniques
-   **Quality Attribute Workshops (QAW)**: Sessions to prioritize NFRs.
-   **Scenario-Based Elicitation**: Creating "Architecture Scenarios" (Source, Stimulus, Artifact, Environment, Response, Response Measure).

### Application in Go (Golang)

In a Go project, requirements often translate directly into benchmark tests or specific library choices.

#### 1. Performance Requirements
If the elicitation reveals a need for high concurrency:
*   *Go Context*: Architect decides to use Goroutines and Channels, or potentially a worker pool pattern.
*   *Verification*: Writing Go Benchmark tests (`func BenchmarkXxx`) to prove the requirement is met.

#### 2. Scalability Requirements
If the system must scale horizontally:
*   *Go Context*: Designing stateless HTTP handlers using `net/http` or `Gin`, ensuring no local session state is stored in memory.

## Interview Questions

### Q: How do you handle conflicting requirements from different stakeholders?
**A:** I start by making the conflict visible using a trade-off matrix. For example, if Marketing wants "Fastest Time to Market" but Security wants "Bank-Grade Encryption", I explain that we can't maximize both. I facilitate a negotiation session where we agree on the "North Star" metrics for the project, prioritizing the NFRs that drive the core business value.

### Q: What is the difference between Functional and Non-Functional Requirements?
**A:** Functional Requirements define **what** the system does (features, inputs, outputs). Non-Functional Requirements (Quality Attributes) define **how** the system performs (speed, security, reliability). As an architect, NFRs are my primary concern because they dictate the architectural structure.

### Q: How do you quantify "Scalability"?
**A:** Scalability isn't a binary "yes/no". I quantify it by defining the **Load Parameter** (e.g., concurrent users, data volume) and the **Resource Metric** (CPU, RAM). A scalable system is one where adding resources (cost) results in a proportional increase in capacity. I look for the "Scaling Factor" (ideal is 1.0, linear).
