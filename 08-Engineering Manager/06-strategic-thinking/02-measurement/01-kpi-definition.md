# KPI Definition

## Summary
Key Performance Indicators (KPIs) are quantifiable measurements used to gauge a team's long-term performance and health. Unlike OKRs (which track ambitious, temporary goals), KPIs track the steady-state "heartbeat" of the engineering organization. Good KPIs should drive the right behaviors and avoid "Goodhart's Law" (where a measure becomes a target and ceases to be a good measure).

## Detailed Explanation

### 1. KPI vs. OKR
*   **OKRs (Objectives & Key Results)**: "Change the business." (e.g., "Launch new product X"). They are temporary and aspirational.
*   **KPIs (Key Performance Indicators)**: "Run the business." (e.g., "Uptime 99.9%"). They are permanent and operational.
*   *Analogy*: OKR is the destination on the GPS; KPIs are the dashboard gauges (Speed, Fuel, RPM).

### 2. Leading vs. Lagging Indicators
*   **Lagging**: Tells you what happened (Output). E.g., "Revenue," "Bugs Reported."
*   **Leading**: Predicts what *will* happen (Input). E.g., "Code Review Velocity," "Daily Active Users." Managers should focus on leading indicators to influence outcomes.

### 3. Categories of Engineering KPIs
*   **Velocity**: Story points completed (use with caution).
*   **Quality**: Change Failure Rate, Bug Detection Rate.
*   **Efficiency**: Cycle Time (Start to Finish).
*   **Stability**: Uptime, MTTR.

### 4. Goodhart's Law
> "When a measure becomes a target, it ceases to be a good measure."
*   If you measure "Lines of Code," devs will write verbose code.
*   If you measure "Number of Commits," devs will make tiny commits.
*   *Solution*: Use paired metrics (e.g., Velocity + Quality) to balance incentives.

## Go Code Example: KPI Dashboard Aggregator
This example models a system that ingests raw data points and aggregates them into high-level KPIs, flagging any that are "At Risk."

```go
package main

import (
	"fmt"
)

// KPIStatus enum
type KPIStatus string

const (
	Healthy KPIStatus = "GREEN"
	AtRisk  KPIStatus = "YELLOW"
	Critical KPIStatus = "RED"
)

// KPI Definition
type KPI struct {
	Name      string
	Value     float64
	Target    float64
	Threshold float64 // How much deviation is allowed before flagging
	IsHigherBetter bool
}

// Evaluate checks the KPI against its target
func (k KPI) Evaluate() KPIStatus {
	var deviation float64
	if k.IsHigherBetter {
		if k.Value >= k.Target {
			return Healthy
		}
		deviation = k.Target - k.Value
	} else {
		// Lower is better (e.g., Latency, Bugs)
		if k.Value <= k.Target {
			return Healthy
		}
		deviation = k.Value - k.Target
	}

	if deviation <= k.Threshold {
		return AtRisk
	}
	return Critical
}

func main() {
	dashboard := []KPI{
		{Name: "System Uptime (%)", Value: 99.95, Target: 99.99, Threshold: 0.05, IsHigherBetter: true},
		{Name: "Avg Cycle Time (Days)", Value: 4.5, Target: 3.0, Threshold: 2.0, IsHigherBetter: false},
		{Name: "Customer Bugs (Count)", Value: 12.0, Target: 5.0, Threshold: 5.0, IsHigherBetter: false},
	}

	fmt.Println("--- Engineering KPI Dashboard ---")
	for _, metric := range dashboard {
		status := metric.Evaluate()
		icon := ""
		switch status {
		case Healthy:
			icon = "✅"
		case AtRisk:
			icon = "⚠️"
		case Critical:
			icon = "🔥"
		}
		fmt.Printf("%s %s: %.2f (Target: %.2f) - %s\n", icon, metric.Name, metric.Value, metric.Target, status)
	}
}
```

## Interview Questions

### Q: "What is the danger of using Velocity as a performance KPI?"
**A:**
*   **Story Points are relative**, not absolute. They vary by team.
*   It encourages **point inflation** (calling a 1-point task a 3-point task).
*   It discourages **quality** (rushing to finish points).
*   *Better Approach*: Use velocity only for capacity planning, not for judging performance.

### Q: "If you could only track 3 KPIs for your team, what would they be and why?"
**A:**
*   1. **Cycle Time** (Speed): How fast do we deliver value?
*   2. **Change Failure Rate** (Quality): How often do we break things?
*   3. **Employee Net Promoter Score (eNPS)** (People): Is the team burnt out?
*   *Why*: This covers the "Iron Triangle" of Speed, Quality, and People.

### Q: "Explain Leading vs. Lagging indicators with an example."
**A:**
*   **Lagging**: "We missed the release deadline." (Too late to fix).
*   **Leading**: "PRs are sitting unreviewed for 3 days." (Predicts we will miss the deadline). I watch the PR queue to prevent the slip.
