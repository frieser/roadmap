# Company Culture

## Summary
Company Culture is the "shadow operating system" of an organization. While an Engineering Manager builds *Team* Culture (micro), they must also align with the broader *Company* Culture (macro). Understanding this alignment—or misalignment—is crucial for navigating decisions, hiring, and survival.

## Detailed Explanation

### 1. Types of Cultures (The Westrum Topology)
Sociologist Ron Westrum defined three organizational types:
*   **Pathological (Power-oriented)**: Information is hidden, messengers are shot, responsibilities are shirked, cooperation is discouraged.
*   **Bureaucratic (Rule-oriented)**: Rules are paramount, positions are fixed, cross-department cooperation is modest.
*   **Generative (Performance-oriented)**: Risks are shared, failure leads to inquiry, new ideas are welcomed. *This is the goal for high-performing tech companies.*

### 2. Artifacts, Espoused Values, and Assumptions
(Edgar Schein's Model)
*   **Artifacts**: Visible things (Open plan office? Dress code? Remote policy?).
*   **Espoused Values**: What they *say* on the website ("We value integrity").
*   **Basic Assumptions**: What they *do* when nobody is looking. (Do they promote the jerk who hits revenue targets?). The gap between Values and Assumptions is where cynicism breeds.

### 3. Subcultures
Engineering often has a different subculture than Sales.
*   **Sales**: Short-term, quota-driven, extroverted.
*   **Eng**: Long-term, quality-driven, introverted.
*   *Role of EM*: Be the bridge/translator between these subcultures.

## Go Code Example: Culture Fit Scorer
This example helps analyze if a candidate or a proposed initiative fits the specific dimensions of the company culture.

```go
package main

import (
	"fmt"
)

type CultureDimension string

const (
	SpeedVsQuality      CultureDimension = "Speed vs Quality"
	AutonomyVsControl   CultureDimension = "Autonomy vs Control"
	RiskVsStability     CultureDimension = "Risk vs Stability"
)

type CompanyProfile struct {
	Dimensions map[CultureDimension]int // 1 (Left) to 10 (Right)
}

func AnalyzeFit(company CompanyProfile, candidateScore int, dim CultureDimension) string {
	companyScore := company.Dimensions[dim]
	diff := companyScore - candidateScore
	if diff < 0 {
		diff = -diff
	}

	if diff <= 2 {
		return "✅ High Fit"
	} else if diff <= 4 {
		return "⚠️  Moderate Fit (Friction likely)"
	}
	return "❌ Low Fit (Culture Clash)"
}

func main() {
	// Startup Mode: High Speed, High Risk
	startup := CompanyProfile{
		Dimensions: map[CultureDimension]int{
			SpeedVsQuality:    2, // 1=Speed, 10=Quality
			AutonomyVsControl: 2, // 1=Autonomy, 10=Control
			RiskVsStability:   1, // 1=Risk, 10=Stability
		},
	}

	// Candidate: meticulous, slow, risk-averse (Ex-Banking)
	candidateVal := 9 // Prefers Quality/Stability

	fmt.Println("--- Culture Fit Analysis ---")
	fmt.Printf("Dimension: %s\n", SpeedVsQuality)
	fmt.Printf("Company Score: %d | Candidate Score: %d\n", startup.Dimensions[SpeedVsQuality], candidateVal)
	fmt.Println("Result:", AnalyzeFit(startup, candidateVal, SpeedVsQuality))
}
```

## Interview Questions

### Q: "How do you handle a mismatch between your personal values and the company culture?"
**A:**
*   **Assess the Gap**: Is it a difference in *style* (e.g., they like meetings, I don't) or *ethics* (e.g., they lie to customers)?
*   **Adapt or Exit**: If it's style, I adapt. If it's ethics, I leave.
*   **Shield the Team**: As a manager, I try to create a "micro-culture" of safety within my team, buffering them from the toxicity above (to a limit).

### Q: "Describe the culture of your current company. What would you change?"
**A:**
*   **Focus**: Be objective. "We are very consensus-driven (Bureaucratic), which ensures buy-in but slows down decision making significantly. I would introduce 'Disagree and Commit' to speed up execution."

### Q: "How do you maintain culture in a remote-first company?"
**A:**
*   **Intentionality**: Culture doesn't happen at the water cooler anymore. It happens in how we write documentation, how we handle async communication, and how we respect time zones.
*   **Over-communication**: In remote, silence is interpreted as anxiety. We must over-communicate praise and status.
