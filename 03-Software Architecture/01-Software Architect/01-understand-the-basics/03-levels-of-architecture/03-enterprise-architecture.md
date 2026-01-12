---
---

## Summary
Enterprise Architecture (EA) provides the strategic technical direction for the entire organization. It focuses on aligning IT strategy with business goals, standardizing technologies across teams, and optimizing the overall IT portfolio to reduce complexity and cost.

## Detailed Explanation

Enterprise Architecture is the "Bird's Eye" view. It is less about "How do I build this app?" and more about "How do we build apps at this company?".

### Key Responsibilities
1.  **Strategic Alignment**: Ensuring technology investments support business objectives (e.g., "We are moving to Cloud-First to improve agility").
2.  **Standardization**: Defining the "Paved Road" or "Golden Path"—the recommended set of tools, languages, and platforms (e.g., "We use AWS, Go for backends, React for frontends").
3.  **Governance**: Establishing policies for security, compliance, and data privacy.
4.  **Portfolio Management**: Deciding when to retire legacy systems ("Sunsetting") and when to adopt new technologies.

### The "Service Chassis"
A key concept in EA is the **Service Chassis**: a standardized framework or library that handles cross-cutting concerns (logging, config, auth) so teams can focus on business logic.

### Application in Go (Golang)

In a large organization using Go, the Enterprise Architect's role involves creating an ecosystem where Go teams can move fast.

#### 1. The Standard Go Library (Internal)
EA often mandates or provides a shared internal library (e.g., `github.com/mycompany/kit`) that wraps standard libraries to enforce company policies.
*   *Example*: A wrapper around `zap` logger that automatically injects the standard TraceID and SpanID for observability.

#### 2. Platform Engineering
EA promotes building internal tools in Go to standardize workflows.
*   *CLI Tools*: A `myco-cli` written in Go that scaffolds new microservices with the correct folder structure, Dockerfile, and CI pipeline configuration.
*   *Terraform Providers*: Custom Go-based Terraform providers to manage internal infrastructure resources.

#### 3. Polyglot Strategy
EA decides where Go fits. A common pattern:
*   **Go**: For high-throughput, low-latency backend services and infrastructure tools.
*   **Python**: For Data Science and AI/ML workloads.
*   **Node/TS**: For Backend-for-Frontend (BFF) layers closer to the UI.

```go
// Example: Enterprise Standard "Service Chassis" usage
// All teams MUST use this bootstrap code to ensure compliance.

import "github.com/mycompany/kit/service"

func main() {
    // Standard service initialization handles:
    // - Loading config from Vault
    // - Setting up OpenTelemetry tracing
    // - Configuring standard JSON logging
    // - Registering health check endpoints
    svc := service.New("payment-service")
    
    svc.Run() // Blocks until SIGTERM
}
```

## Interview Questions

### Q: How do you balance team autonomy with enterprise standardization?
**A:** I use the concept of the **"Golden Path"** (or Paved Road). We provide a supported, high-quality set of tools/platforms (The Golden Path). If teams stay on it, they get free support, tooling, and easy compliance. Teams *can* go off-road (Autonomy), but they own the full burden of operation, security compliance, and maintenance. Most teams choose the Golden Path because it's the path of least resistance.

### Q: What is your approach to retiring legacy systems (Strangler Fig Pattern)?
**A:** We don't do "Big Bang" rewrites. We use the **Strangler Fig Pattern**. We place a proxy/router in front of the legacy system. We build new features in modern microservices (e.g., in Go) and route traffic for those features to the new services. Over time, we migrate existing functionality piece by piece until the legacy system is effectively "strangled" and can be decommissioned.

### Q: How does Enterprise Architecture support Digital Transformation?
**A:** EA provides the roadmap. It identifies the "As-Is" architecture (often siloed, manual, slow) and defines the "To-Be" architecture (cloud-native, automated, agile). EA then creates the transition plan, prioritizing changes that unlock the most business value (e.g., "First, we containerize apps to enable cloud migration, then we break down the monolith to enable faster release cycles").
