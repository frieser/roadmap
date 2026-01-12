# Technology Adoption

## Summary
Technology Adoption is the process of introducing a new language, framework, or tool to the organization. EMs must guard against "Resume Driven Development" (using cool tech just to get hired elsewhere) while ensuring the stack doesn't become obsolete. The key is **Standardization** vs. **Innovation**.

## Detailed Explanation

### 1. The Technology Radar
*   Categorize tech into rings:
    *   **Adopt**: Safe, standard (e.g., Go, Postgres).
    *   **Trial**: Use for non-critical pilots (e.g., Rust).
    *   **Assess**: Researching (e.g., Bun).
    *   **Hold**: Do not use / Deprecate (e.g., MongoDB 3.x).

### 2. Innovation Tokens (Dan McKinley)
*   "You get 3 innovation tokens. Choose wisely."
*   Using boring tech (Postgres, Rails, Java) saves tokens for where it matters (e.g., Custom AI Model).
*   Don't spend tokens on your build system or database unless you are Google.

### 3. The Pilot Project
*   Never roll out new tech globally.
*   Pick a low-risk, isolated project.
*   Measure results. "Did it actually speed us up?"

## Go Code Example: Tech Stack Validator
Checks if a proposed project uses approved technologies.

```go
package main

import (
	"fmt"
)

var TechRadar = map[string]string{
	"Go":       "Adopt",
	"Postgres": "Adopt",
	"Rust":     "Trial",
	"Haskell":  "Hold", // Too hard to hire for
}

func ApproveStack(languages []string) bool {
	for _, lang := range languages {
		status := TechRadar[lang]
		if status == "Hold" {
			fmt.Printf("❌ REJECTED: %s is on Hold.\n", lang)
			return false
		}
		if status == "Trial" {
			fmt.Printf("⚠️  WARNING: %s is Trial. Requires Architecture Review.\n", lang)
		}
	}
	fmt.Println("✅ Stack Approved.")
	return true
}

func main() {
	proposal := []string{"Go", "Postgres", "Haskell"}
	ApproveStack(proposal)
}
```

## Interview Questions

### Q: "A Senior Engineer wants to rewrite the backend in Rust. Thoughts?"
**A:**
*   **Why?**: Is it for memory safety/performance, or because it's cool?
*   **Hiring**: Can we hire Rust devs?
*   **Bus Factor**: If she leaves, who maintains it?
*   **Decision**: "Prove it. Rewrite one small, high-performance microservice. If it works, we discuss more."

### Q: "How do you balance standardization with autonomy?"
**A:**
*   **Golden Path**: "If you use the standard stack (Go/AWS), you get free tooling/CI/Support. If you go off-road (Node/Azure), you are on your own."
*   Most teams will choose the path of least resistance.

### Q: "What is 'Resume Driven Development'?"
**A:**
*   Engineers picking tech that looks good on their CV (e.g., Kubernetes for a static site) rather than what solves the business problem.
*   *Mitigation*: Focus on "Boring Technology" for core business logic.
