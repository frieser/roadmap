# Executive Summaries

## Summary
An Executive Summary is a brief document (or section) that distills complex information into the key points needed for decision-making. Executives are "time-poor and information-rich." Your job is to respect their time by putting the conclusion first (BLUF: Bottom Line Up Front).

## Detailed Explanation

### 1. The BLUF Method
*   **Bottom Line Up Front**: Start with the ask or the conclusion.
*   *Bad*: "We looked at Postgres, then Mongo, then Redis..."
*   *Good*: "Recommendation: Migrate to Postgres to save $10k/month."

### 2. The Structure
1.  **The Context**: 1 sentence on why we are talking.
2.  **The Problem**: What is broken/risky?
3.  **The Solution**: What do you propose?
4.  **The Ask**: What do you need (Money, Approval, Headcount)?
5.  **The Risks**: What if we do nothing?

### 3. Know Your "Currency"
*   **CEO Currency**: Growth, Market Share.
*   **CFO Currency**: Cost, ROI, Risk.
*   **CTO Currency**: Velocity, Stability, Innovation.
*   *Tailor the summary to the reader.*

## Go Code Example: Summary Generator
This tool truncates a long technical report into a BLUF format.

```go
package main

import (
	"fmt"
)

type TechReport struct {
	Title       string
	FullDetails string
	Cost        float64
	Risk        string
	Recommendation string
}

func (r TechReport) GenerateBLUF() string {
	return fmt.Sprintf(`
📋 **EXECUTIVE SUMMARY: %s**

**Recommendation**: %s
**Financial Impact**: $%.0f
**Key Risk**: %s

---
*Full details attached below...*
`, r.Title, r.Recommendation, r.Cost, r.Risk)
}

func main() {
	report := TechReport{
		Title: "Database Migration Strategy",
		FullDetails: "We analyzed 5 different vendors. Oracle is too expensive. MySQL is good but...",
		Cost: 50000,
		Risk: "2 hours planned downtime required.",
		Recommendation: "Migrate to Amazon Aurora (Postgres Compatible).",
	}

	fmt.Println(report.GenerateBLUF())
}
```

## Interview Questions

### Q: "How do you deliver bad news to an executive?"
**A:**
*   **Fast**: Don't wait. Bad news does not get better with age.
*   **Direct**: "We found a security vulnerability."
*   **With a Plan**: "We have patched it. We are auditing for others. Here is the draft communication for customers."
*   *Never bring a problem without a proposed solution (or at least a next step).*

### Q: "An exec sends you a 2am email asking about a minor bug. What do you do?"
**A:**
*   **Don't panic**: They are just clearing their inbox.
*   **Reply during work hours**: (Unless it's SEV1). Training them that you are awake at 2am leads to burnout.
*   **Concise reply**: "Confirmed bug. Priority Low. Added to backlog. Will be fixed in next sprint."

### Q: "How do you explain technical debt to a CFO?"
**A:**
*   **Financial Analogy**: "It's like a high-interest credit card. We took a loan to ship the MVP fast (which was good!). Now we are paying interest (slower dev speed). If we don't pay down the principal, the interest will consume our entire budget."
