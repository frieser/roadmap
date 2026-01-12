## Summary
Risk Management is the proactive process of identifying, analyzing, and responding to project risks. A risk is an uncertain event or condition that, if it occurs, has a positive or negative effect on project objectives. The goal is to minimize the probability/impact of negative risks and maximize opportunities.

## Detailed Explanation
Risks are not problems; problems are risks that have already happened. Management is about keeping risks as risks.

### The Risk Cycle
1.  **Identify**: Brainstorming "What could go wrong?" (e.g., Tech debt, team turnover, 3rd party API failure).
2.  **Analyze**: Determine Probability (Likelihood) and Impact (Severity).
    *   *Risk Score = Probability x Impact*
3.  **Prioritize**: Focus on High Probability / High Impact items.
4.  **Response Planning**:
    *   **Avoid**: Change plan to eliminate risk (e.g., choose a mature library instead of beta).
    *   **Mitigate**: Reduce probability or impact (e.g., adding caching to mitigate slow API).
    *   **Transfer**: Shift responsibility (e.g., insurance, outsourcing).
    *   **Accept**: Do nothing but set aside contingency buffer.
5.  **Monitor**: Re-evaluate risks regularly.

## Go Code Example
A Risk Register implementation that calculates risk scores and suggests mitigation strategies based on severity.

```go
package main

import (
	"fmt"
	"sort"
)

type ImpactLevel int
type ProbabilityLevel int

const (
	LowImpact    ImpactLevel = 1
	MediumImpact ImpactLevel = 2
	HighImpact   ImpactLevel = 3
	CriticalImpact ImpactLevel = 4

	LowProb    ProbabilityLevel = 1
	MediumProb ProbabilityLevel = 2
	HighProb   ProbabilityLevel = 3
)

type Risk struct {
	Name        string
	Impact      ImpactLevel
	Probability ProbabilityLevel
}

// Score calculates the Risk Severity
func (r Risk) Score() int {
	return int(r.Impact) * int(r.Probability)
}

func (r Risk) MitigationStrategy() string {
	score := r.Score()
	switch {
	case score >= 9:
		return "CRITICAL: Immediate Action / Plan B required"
	case score >= 4:
		return "HIGH: Active Mitigation and Monitoring"
	default:
		return "LOW: Accept / Watch List"
	}
}

func main() {
	register := []Risk{
		{"Server Crash", CriticalImpact, LowProb},      // 4 * 1 = 4
		{"API Rate Limit", MediumImpact, HighProb},     // 2 * 3 = 6
		{"Key Dev Sick", HighImpact, MediumProb},       // 3 * 2 = 6
		{"Typos in UI", LowImpact, MediumProb},         // 1 * 2 = 2
		{"Vendor Bankrupt", CriticalImpact, MediumProb},// 4 * 2 = 8
	}

	// Sort by Risk Score (Descending)
	sort.Slice(register, func(i, j int) bool {
		return register[i].Score() > register[j].Score()
	})

	fmt.Printf("%-20s | %-5s | %s\n", "Risk", "Score", "Strategy")
	fmt.Println("---------------------------------------------------------")
	for _, r := range register {
		fmt.Printf("%-20s | %-5d | %s\n", r.Name, r.Score(), r.MitigationStrategy())
	}
}
```

## Interview Questions
**Q: How do you identify risks in a new project?**
**A:** I conduct a "Pre-mortem" session with the team where we assume the project has failed and ask "Why did it fail?" This psychological trick releases people to voice concerns they might otherwise hide. I also review "Lessons Learned" from previous similar projects.

**Q: What is the difference between a Risk and an Issue?**
**A:** A Risk is a future uncertainty (it *might* happen). An Issue is a current problem (it *has* happened). You *manage* risks to prevent them from becoming issues. You *resolve* issues.

**Q: Example of a technical risk you managed?**
**A:** "We identified a risk that our 3rd party payment provider might not support the new currency we were launching. We mitigated this by building an abstraction layer (Adapter Pattern) early, which allowed us to swap providers quickly if the risk materialized (which it did)."
