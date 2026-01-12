# Competitive Analysis

## Summary
Competitive Analysis is the practice of evaluating the strengths and weaknesses of current and potential competitors. For EMs, this isn't just about product features, but about **Technical Intelligence**: How fast do they ship? What is their uptime? What technologies do they use? This informs your own "Build vs. Buy" and architectural strategies.

## Detailed Explanation

### 1. Technical Benchmarking
*   **Velocity**: Look at their changelog. Are they shipping weekly or quarterly?
*   **Performance**: Run Lighthouse scores on their site. Check their API latency.
*   **Reliability**: Check their status page history. Do they go down often?

### 2. Feature Parity vs. Differentiation
*   **Table Stakes**: Features you *must* have to even play (e.g., Login, Reset Password). Copy these standard patterns.
*   **Differentiators**: The "Killer Features." Spend your innovation tokens here.

### 3. Hiring Intelligence
*   Look at their Job Boards.
*   If they are hiring 10 "Data Engineers," they are likely building a new Analytics product.
*   If they are hiring "Security Compliance Officers," they are trying to move Enterprise/Upmarket.

## Go Code Example: Feature Matrix Comparator
This example compares your product against competitors to identify gaps and opportunities.

```go
package main

import (
	"fmt"
)

type Product struct {
	Name     string
	Features map[string]bool
	Price    float64
}

func Compare(myProduct Product, competitors []Product) {
	fmt.Printf("--- Competitive Analysis for %s ---\n", myProduct.Name)
	
	allFeatures := make(map[string]bool)
	// Collect all known market features
	for f := range myProduct.Features {
		allFeatures[f] = true
	}
	for _, c := range competitors {
		for f := range c.Features {
			allFeatures[f] = true
		}
	}

	for feature := range allFeatures {
		hasMine := myProduct.Features[feature]
		
		missingCount := 0
		for _, c := range competitors {
			if !c.Features[feature] {
				missingCount++
			}
		}

		if hasMine && missingCount == len(competitors) {
			fmt.Printf("🌟 DIFFERENTIATOR: Only we have [%s]\n", feature)
		} else if !hasMine && missingCount == 0 {
			fmt.Printf("🚨 GAP: Everyone has [%s] except us!\n", feature)
		} else if hasMine {
			fmt.Printf("✅ Parity: We have [%s]\n", feature)
		}
	}
}

func main() {
	us := Product{
		Name: "OurApp",
		Features: map[string]bool{
			"SSO": true, "API": true, "Dark Mode": false,
		},
	}

	them := []Product{
		{
			Name: "Competitor X",
			Features: map[string]bool{
				"SSO": true, "API": true, "Dark Mode": true, "Mobile App": true,
			},
		},
		{
			Name: "Competitor Y",
			Features: map[string]bool{
				"SSO": true, "API": false, "Dark Mode": true,
			},
		},
	}

	Compare(us, them)
}
```

## Interview Questions

### Q: "Your competitor just launched a feature we don't have. What is your reaction?"
**A:**
*   **Don't Panic**.
*   **Evaluate**: Does this change the game? Do our customers actually care?
*   **Strategic Response**:
    *   *Ignore*: If it's a distraction.
    *   *Fast Follow*: If it's becoming table stakes, build a "good enough" version quickly.
    *   *Leapfrog*: Build something better that makes their feature irrelevant.

### Q: "How can you tell if a competitor's engineering team is struggling?"
**A:**
*   **Signs**:
    *   Changelog silence (no updates for months).
    *   Regressions (old bugs reappearing).
    *   High turnover (LinkedIn insights).
    *   This signals an opportunity for us to capture market share by being reliable.

### Q: "Should we copy a competitor's architecture (e.g., they use GraphQL, so should we)?"
**A:**
*   **No**. You are looking at their *solution*, not their *problem*.
*   They might regret using GraphQL. Or they might have 500 engineers to support it, while we have 5.
*   Always evaluate technology based on *our* constraints and *our* team's skills.
