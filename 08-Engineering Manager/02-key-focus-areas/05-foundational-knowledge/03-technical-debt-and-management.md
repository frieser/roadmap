## Summary
Technical debt describes the implied cost of additional rework caused by choosing an easy (limited) solution now instead of using a better approach that would take longer. Like financial debt, technical debt incurs "interest" in the form of extra effort required to maintain code, and it must eventually be "paid down" via refactoring or it will bankrupt the project's velocity.

## Detailed Explanation
### Types of Technical Debt
*   **Deliberate/Prudent**: "We must ship now, we will fix the hard-coded limit next sprint." (Requires tracking).
*   **Inadvertent/Reckless**: "I didn't know how to layer this properly so I just put everything in main." (Needs education).
*   **Bit Rot**: Code that was once good but has become obsolete as libraries or requirements changed.

### Managing Debt
*   **Visibility**: Track debt in the backlog (e.g., "Tech Debt" tag).
*   **Allocation**: Dedicate % of sprint capacity (e.g., 20%) to refactoring.
*   **Boy Scout Rule**: Always leave the code behind in a better state than you found it.

### Go Code Example: Refactoring Debt
Here is an example of code with technical debt (hard dependencies, magic numbers) refactored into clean, manageable code.

```go
package main

import "fmt"

// --- BAD CODE (High Debt) ---
// Hard to test, hard to change config, mixed concerns.
func ProcessPaymentBad(amount float64) {
	if amount > 1000 { // Magic number
		fmt.Println("Large payment, checking fraud...") // Side effect in logic
	}
	// Hardcoded DB connection
	fmt.Printf("Connecting to prod-db-01... Paying %.2f\n", amount)
}

// --- REFACTORED CODE (Paid Debt) ---

type PaymentConfig struct {
	MaxTransferLimit float64
	DBConnectionStr  string
}

type FraudDetector interface {
	Check(amount float64) bool
}

type PaymentProcessor struct {
	Config   PaymentConfig
	Detector FraudDetector
}

func (p *PaymentProcessor) Process(amount float64) error {
	if amount > p.Config.MaxTransferLimit {
		if p.Detector.Check(amount) {
			return fmt.Errorf("fraud detected")
		}
	}
	fmt.Printf("Connecting to %s... Paying %.2f\n", p.Config.DBConnectionStr, amount)
	return nil
}

// Simple implementation for example
type SimpleFraudDetector struct{}

func (s *SimpleFraudDetector) Check(amount float64) bool {
	fmt.Println("Running fraud check...")
	return false
}

func main() {
	// Explicit configuration
	config := PaymentConfig{
		MaxTransferLimit: 1000.0,
		DBConnectionStr:  "prod-db-01",
	}
	
	processor := &PaymentProcessor{
		Config:   config,
		Detector: &SimpleFraudDetector{},
	}
	
	processor.Process(1500.0)
}
```

## Interview Questions
**Q: How do you explain technical debt to non-technical stakeholders?**
**A:** I use the kitchen metaphor: If you run a restaurant and never clean the kitchen (wash dishes, organize tools) to serve food faster, initially you go fast. But eventually, the mess slows you down, hygiene risks appear, and you can't cook anything. Technical debt is that mess; we need time to clean up so we can keep serving features quickly.

**Q: How do you prioritize technical debt against new features?**
**A:** I treat technical debt as a risk. High-interest debt (code that is modified frequently and is fragile) gets priority. I often advocate for a 20% capacity reservation for "engineering health" tasks in every sprint, or dedicated "cool-down" sprints after major releases to address accumulated debt.

**Q: Is technical debt always bad?**
**A:** No. Prudent technical debt is a valid business tool to reduce Time-to-Market. If we need to validate a market hypothesis, it's smarter to write "quick and dirty" code. The key is to acknowledge it, track it, and have a plan to pay it down if the hypothesis proves true.
