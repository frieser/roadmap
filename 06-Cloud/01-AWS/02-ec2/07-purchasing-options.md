#AWS #Cloud

---
tags: ['cloud', 'roadmap', 'aws', 'ec2']
---

## Summary
Amazon EC2 provides multiple purchasing options to balance cost, performance, and flexibility. From **On-Demand** pay-as-you-go models to deep-discounted **Spot Instances** and commitment-based **Savings Plans**, understanding these models is critical for effective cloud cost management. This note covers the strategic application of each model and the operational handling of Spot interruptions.

## Detailed Explanation

### 1. On-Demand Instances
*   **Pricing**: Pay for compute capacity by the second (minimum 60 seconds) with no long-term commitment.
*   **Best For**: Low-cost, flexible workloads with unpredictable usage patterns that cannot be interrupted.
*   **Pros**: Complete control over lifecycle, no upfront costs.
*   **Cons**: Highest hourly rate among all options.

### 2. Spot Instances
*   **Deep Dive**: AWS uses Spot Instances to sell spare EC2 capacity at up to a **90% discount** compared to On-Demand prices.
*   **Spot Interruption**: AWS can terminate a Spot instance at any time if they need the capacity back. 
    *   **The 2-Minute Warning**: AWS provides a 2-minute interruption notice via the **Instance Metadata Service (IMDS)** (`http://169.254.169.254/latest/meta-data/spot/instance-action`) and **Amazon EventBridge**.
    *   **Handling Strategies**: 
        *   **Graceful Shutdown**: Drain connections from load balancers and stop accepting new tasks.
        *   **Checkpointing**: Save state to S3 or DynamoDB to resume work later from the last known point.
        *   **Capacity Rebalancing**: Use EC2 Auto Scaling "Capacity Rebalance" to proactively launch replacement instances when a rebalance recommendation is received.
*   **Best For**: Stateless, fault-tolerant, or flexible applications (e.g., Batch processing, Big Data, CI/CD, Containerized microservices).

### 3. Reserved Instances (RI)
*   **Commitment**: 1 or 3-year commitment for a specific instance configuration (region, instance family, OS).
*   **Types**:
    *   **Standard**: Up to 75% savings. Rigid (cannot change instance family).
    *   **Convertible**: Up to 54% savings. Allows changing instance family, OS, and tenancy.
*   **Best For**: Known, steady-state workloads (e.g., production databases).

### 4. Savings Plans
*   **Concept**: Commit to a consistent amount of usage (measured in $/hour) for 1 or 3 years.
*   **Compute Savings Plans**: Most flexible; automatically applies across EC2 (any region, family), Fargate, and Lambda.
*   **EC2 Instance Savings Plans**: Less flexible but higher savings; applies to a specific instance family in a region.
*   **Best For**: Modern architectures needing flexibility across compute services and regions.

### 5. Cost Optimization Strategies
*   **The "Pyramid" Strategy**: 
    1.  **Baseline**: Cover the 24/7 "floor" of your compute usage with **Savings Plans** or **RIs**.
    2.  **Elastic Scaling**: Use **On-Demand** for the variable, unpredictable part of your workload.
    3.  **Flexible Workloads**: Offload batch processing and non-critical scaling to **Spot Instances**.
*   **Mixed Instance Groups**: Use Auto Scaling Groups with a "Mixed Instances Policy" to combine Spot and On-Demand to maximize availability while minimizing cost.
*   **Lifecycle Management**: Schedule non-production environments (Dev/Test) to stop during off-hours using AWS Instance Scheduler.

### Practical Implementation (Go)
In Go, you can monitor Spot interruption notices by polling the IMDS endpoint.

```go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	"time"
)

// SpotAction represents the structure returned by IMDS for spot interruptions
type SpotAction struct {
	Action string    `json:"action"`
	Time   time.Time `json:"time"`
}

func monitorSpotInterruption() {
	url := "http://169.254.169.254/latest/meta-data/spot/instance-action"
	
	for {
		resp, err := http.Get(url)
		if err == nil && resp.StatusCode == http.StatusOK {
			var action SpotAction
			if err := json.NewDecoder(resp.Body).Decode(&action); err == nil {
				fmt.Printf("INTERRUPTION DETECTED: %s at %v\n", action.Action, action.Time)
				// Trigger graceful shutdown logic here
				return
			}
			resp.Body.Close()
		}
		time.Sleep(5 * time.Second)
	}
}

func main() {
	fmt.Println("Starting Spot Instance monitor...")
	monitorSpotInterruption()
}
```

## Interview Questions

**Q: How do you handle Spot Instance interruptions in a production environment?**
**A:** Use a combination of **Amazon EventBridge** to listen for interruption notices and **EC2 Instance Metadata (IMDS)** to poll for the 2-minute warning. Implement graceful shutdowns by draining connections from the Target Group and saving application state (checkpointing) to a persistent store like S3 or DynamoDB. Additionally, enable **Capacity Rebalance** in Auto Scaling Groups to proactively replace instances.

**Q: Which is more flexible: a Convertible Reserved Instance or a Compute Savings Plan?**
**A:** A **Compute Savings Plan** is significantly more flexible. While a Convertible RI allows changing instance families, it is still tied to EC2. A Compute Savings Plan automatically applies to any EC2 instance (regardless of region or family), AWS Fargate, and AWS Lambda usage.

**Q: If you have a legacy application that requires a physical server for compliance and specific software licenses, which EC2 option do you choose?**
**A:** A **Dedicated Host**. It provides a physical server fully dedicated to your use, allowing you to comply with regulatory requirements and use existing server-bound software licenses (BYOL).

**Q: A startup has a web application with unpredictable traffic spikes. They want to minimize cost but cannot tolerate any downtime. What is the best strategy?**
**A:** Use **On-Demand** instances for the variable spikes to ensure zero interruption. For the minimum baseline traffic that is always present, purchase a **Savings Plan** to reduce the cost of those "always-on" instances.

**Q: What happens to a Spot Instance if the Spot Price exceeds your maximum bid?**
**A:** The instance will be interrupted. AWS will provide a 2-minute notice before terminating, stopping, or hibernating the instance (depending on your configuration). Note: Most users now use the default "On-Demand Price" as the max bid to simplify management.
