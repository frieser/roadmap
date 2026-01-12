# Quality Metrics

## Summary
Quality metrics quantify the reliability and stability of the software delivery process. The industry gold standard are the **DORA Metrics** (DevOps Research and Assessment), which correlate engineering performance with organizational success. Measuring quality prevents the "move fast and break things" anti-pattern.

## Detailed Explanation

### 1. The DORA Metrics (The "Four Keys")
1.  **Deployment Frequency**: How often do we release to production? (Speed).
    *   *Elite*: On-demand (multiple per day).
2.  **Lead Time for Changes**: Time from code commit to code running in production. (Speed).
    *   *Elite*: Less than one hour.
3.  **Change Failure Rate (CFR)**: What % of deployments result in a failure (hotfix/rollback)? (Quality).
    *   *Elite*: 0-15%.
4.  **Mean Time to Restore (MTTR)**: How long to recover from a failure? (Stability).
    *   *Elite*: Less than one hour.

### 2. Code Quality Metrics
*   **Code Coverage**: % of code executed by tests. (Useful to find gaps, bad as a target).
*   **Cyclomatic Complexity**: How many paths through the code? (Keep it low).
*   **Bug Density**: Bugs found per 1,000 lines of code.

### 3. Process Quality Metrics
*   **Defect Escape Rate**: How many bugs are found by customers vs. found by QA?
    *   *Goal*: Catch 90%+ internally.
*   **PR Pickup Time**: How long a pull request waits for review. Long wait times = context switching and lower quality.

## Go Code Example: DORA Calculator
This example calculates the 4 DORA metrics based on a log of deployments and incidents.

```go
package main

import (
	"fmt"
	"time"
)

type Deployment struct {
	ID        string
	Time      time.Time
	IsSuccess bool
}

type Commit struct {
	Time time.Time
}

// DORAStats holds our metrics
type DORAStats struct {
	DeployCount      int
	FailureCount     int
	TotalLeadTimeHrs float64
	TotalRestoreHrs  float64
}

func CalculateDORA(deploys []Deployment, commits []Commit, incidents []float64) {
	stats := DORAStats{}
	stats.DeployCount = len(deploys)

	// 1. Deployment Frequency
	// (Simplified: just printing count)
	
	// 2. Change Failure Rate
	for _, d := range deploys {
		if !d.IsSuccess {
			stats.FailureCount++
		}
	}
	cfr := (float64(stats.FailureCount) / float64(stats.DeployCount)) * 100

	// 3. Lead Time (Avg time between commit and deploy)
	// Assuming 1-to-1 mapping for simplicity
	for i, c := range commits {
		if i < len(deploys) {
			diff := deploys[i].Time.Sub(c.Time).Hours()
			stats.TotalLeadTimeHrs += diff
		}
	}
	avgLeadTime := stats.TotalLeadTimeHrs / float64(len(commits))

	// 4. MTTR (Mean Time To Restore)
	// incidents slice contains duration of outages in hours
	for _, duration := range incidents {
		stats.TotalRestoreHrs += duration
	}
	mttr := stats.TotalRestoreHrs / float64(len(incidents))

	fmt.Println("--- DORA Metrics Report ---")
	fmt.Printf("1. Deploy Frequency: %d deploys total\n", stats.DeployCount)
	fmt.Printf("2. Lead Time for Changes: %.2f hours\n", avgLeadTime)
	fmt.Printf("3. Change Failure Rate: %.2f%%\n", cfr)
	fmt.Printf("4. Time to Restore Service: %.2f hours\n", mttr)
	
	assessPerformance(avgLeadTime, cfr)
}

func assessPerformance(leadTime, cfr float64) {
	fmt.Print("\nAssessment: ")
	if leadTime < 24 && cfr < 15 {
		fmt.Println("ELITE / HIGH PERFORMER 🏆")
	} else if leadTime < 168 { // 1 week
		fmt.Println("MEDIUM PERFORMER")
	} else {
		fmt.Println("LOW PERFORMER")
	}
}

func main() {
	now := time.Now()
	
	// Simulate Data
	commits := []Commit{
		{now.Add(-2 * time.Hour)},
		{now.Add(-26 * time.Hour)},
		{now.Add(-50 * time.Hour)},
	}
	
	deploys := []Deployment{
		{ID: "v1", Time: now.Add(-1 * time.Hour), IsSuccess: true},
		{ID: "v2", Time: now.Add(-25 * time.Hour), IsSuccess: false}, // Failed
		{ID: "v3", Time: now.Add(-48 * time.Hour), IsSuccess: true},
	}

	incidents := []float64{0.5, 1.0} // Hours to fix the failure

	CalculateDORA(deploys, commits, incidents)
}
```

## Interview Questions

### Q: "How do you improve Lead Time for Changes?"
**A:**
*   **Reduce Batch Size**: Ship smaller changes.
*   **Automate CI/CD**: Remove manual approval gates.
*   **Shift Left**: Test earlier so bugs are caught before the deployment phase.

### Q: "Why is Change Failure Rate (CFR) better than just counting bugs?"
**A:**
*   Counting bugs discourages reporting them.
*   CFR measures the *impact* of the process. If we deploy 10 times and 5 fail, our process is broken. It focuses on the reliability of the release pipeline.

### Q: "What is the trade-off between Speed (Lead Time) and Stability (Change Failure Rate)?"
**A:**
*   **Counter-intuitive**: DORA research shows that Elite performers do *both* better.
*   Going faster (smaller batches) actually *reduces* failure rate because smaller changes are easier to debug and fix. Speed enables stability.
