# Cost Optimization

## Summary
Cost Optimization (FinOps) is the practice of getting the maximum business value for every dollar spent on cloud and technology. It is not just about "spending less," but about "spending efficiently." EMs are responsible for the unit economics of their services.

## Detailed Explanation

### 1. The Cloud Bill Problem
Cloud providers (AWS, Azure) make it easy to spin up resources and forget them ("Zombie Infrastructure"). Costs often grow faster than revenue if unchecked.

### 2. Optimization Levers
*   **Rightsizing**: Moving from `c5.2xlarge` to `c5.large` if CPU usage is only 10%.
*   **Elasticity**: Auto-scaling down at night or weekends.
*   **Purchasing Models**:
    *   *On-Demand*: Expensive, flexible.
    *   *Reserved Instances (RI)*: Commit to 1-3 years for ~40-60% discount.
    *   *Spot Instances*: Bid on spare capacity for ~90% discount (risk of interruption).
*   **Storage Tiers**: Moving rarely accessed S3 data to Glacier.

### 3. Culture of Thrift
*   **Showback/Chargeback**: Show teams how much their services cost. "You built a $50k/month microservice for a $5k/month feature."
*   **Tagging**: You cannot optimize what you cannot measure. Every resource must have an `Owner` and `CostCenter` tag.

## Go Code Example: Cloud Cost Analyzer
This example models a simple cost analyzer that suggests optimizations based on resource utilization.

```go
package main

import (
	"fmt"
)

type Resource struct {
	ID           string
	Type         string // EC2, RDS
	MonthlyCost  float64
	AvgCPU       float64 // Percentage (0-100)
	IsProduction bool
}

func AnalyzeCost(resources []Resource) {
	fmt.Println("--- FinOps Recommendations ---")
	potentialSavings := 0.0

	for _, r := range resources {
		// Rule 1: Idle Resources
		if r.AvgCPU < 5.0 {
			fmt.Printf("🔴 [%s] Idle (CPU %.1f%%). Suggest: TERMINATE. Save $%.2f\n", r.ID, r.AvgCPU, r.MonthlyCost)
			potentialSavings += r.MonthlyCost
			continue
		}

		// Rule 2: Over-provisioned
		if r.AvgCPU < 20.0 {
			savings := r.MonthlyCost * 0.5 // Assume downsizing saves 50%
			fmt.Printf("🟡 [%s] Underutilized (CPU %.1f%%). Suggest: DOWNSIZE. Save $%.2f\n", r.ID, r.AvgCPU, savings)
			potentialSavings += savings
		}

		// Rule 3: Spot Candidates (Non-Prod)
		if !r.IsProduction && r.Type == "EC2" {
			savings := r.MonthlyCost * 0.7 // Spot saves ~70%
			fmt.Printf("🟢 [%s] Non-Prod workload. Suggest: SPOT INSTANCE. Save $%.2f\n", r.ID, savings)
			potentialSavings += savings
		}
	}

	fmt.Printf("\nTotal Potential Monthly Savings: $%.2f\n", potentialSavings)
}

func main() {
	inventory := []Resource{
		{"srv-prod-db", "RDS", 500.0, 45.0, true},   // Optimized
		{"srv-test-01", "EC2", 100.0, 2.0, false},   // Idle zombie
		{"srv-web-01", "EC2", 200.0, 15.0, true},    // Oversized
		{"srv-dev-api", "EC2", 100.0, 40.0, false},  // Good candidate for Spot
	}

	AnalyzeCost(inventory)
}
```

## Interview Questions

### Q: "Your AWS bill doubled last month. How do you investigate?"
**A:**
*   **Cost Explorer**: Drill down by Service (is it EC2? S3?) and by Tag (is it Team A?).
*   **Anomalies**: Look for "runaway" lambda functions or data transfer loops (e.g., downloading from S3 in a different region).
*   **Action**: Identify the owner, stop the bleeding, then implement guardrails (Budgets/Alerts).

### Q: "How do you balance performance vs. cost?"
**A:**
*   **SLA based**: If we are meeting our SLA (e.g., latency < 200ms) with cheaper instances, we should switch.
*   **Unit Economics**: Track "Cost per Transaction" or "Cost per User." If user growth matches cost growth, we are fine. If cost grows faster, we have an architecture problem.

### Q: "What is the 'Spot Instance' trade-off?"
**A:**
*   **Pros**: Cheap (~90% off).
*   **Cons**: Can be terminated by AWS with 2 minutes notice.
*   **Use Case**: Stateless services, batch processing, CI/CD runners. NEVER for primary databases.
