## Summary
Code review is a systematic examination of computer source code by someone other than the original author. Its primary goals are to improve code quality, ensure consistency, share knowledge among team members, and prevent bugs. It is one of the most effective tools for maintaining a healthy codebase and engineering culture.

## Detailed Explanation
### Best Practices for Reviewers
*   **Be Constructive**: Comment on the code, not the person. Use questions ("Would X be more efficient?") rather than commands ("Change this to X").
*   **Focus on Logic**: Don't nitpick formatting if a linter can handle it. Focus on architectural fit, bugs, and edge cases.
*   **Timeliness**: Review code quickly to unblock peers (e.g., within 24 hours).

### Best Practices for Authors
*   **Small PRs**: Keep pull requests small and focused on a single concern.
*   **Context**: Provide a clear description and testing steps in the PR.
*   **Self-Review**: Review your own code before assigning it to others.

### Go Code Example: Automated Linter Rules
In Go, much of the "style" debate in code review is solved by `gofmt` and linters. An Engineering Manager might define a struct to represent the team's automated review policy.

```go
package review

import "fmt"

// Rule represents a code quality standard
type Rule struct {
	Name        string
	Severity    string // "Blocker", "Warning", "Nit"
	Description string
}

// LinterConfig represents the team's agreed standards
type LinterConfig struct {
	MaxLineLength int
	RequiredTests bool
	Rules         []Rule
}

// CheckCode simulates running automated checks
func (c *LinterConfig) CheckCode(prSizeLines int, hasTests bool) []string {
	var feedback []string

	if prSizeLines > 400 {
		feedback = append(feedback, "[Warning] PR is too large (>400 lines). Consider splitting.")
	}

	if c.RequiredTests && !hasTests {
		feedback = append(feedback, "[Blocker] PR is missing tests.")
	}

	return feedback
}

func main() {
	teamConfig := LinterConfig{
		MaxLineLength: 80,
		RequiredTests: true,
		Rules: []Rule{
			{"NoPanic", "Blocker", "Do not use panic() in production code"},
			{"ErrCheck", "Blocker", "Always check returned errors"},
		},
	}

	// Simulating a PR Review
	comments := teamConfig.CheckCode(500, false)

	for _, c := range comments {
		fmt.Println(c)
	}
}
```

## Interview Questions
**Q: What do you look for in a code review?**
**A:** I look for correctness (does it do what it should?), maintainability (is it readable and modular?), security (are inputs validated?), and test coverage (are the new changes tested?). I also check if the architecture aligns with the system's design patterns.

**Q: How do you handle a disagreement in a code review?**
**A:** If there's a stalemate, I suggest hopping on a quick call to discuss, as text can be misinterpreted. If the disagreement is about style, I defer to the style guide. If it's architectural, I weigh the pros and cons; if the author's approach is valid and safe, I generally bias towards approving to maintain momentum (disagree and commit), unless it introduces significant debt.

**Q: Why are small Pull Requests important?**
**A:** Small PRs are easier to review thoroughly, reducing the chance of bugs slipping through. They are merged faster, reducing merge conflicts and context switching. Large PRs often get "LGTM" (Looks Good To Me) stamps because reviewers are overwhelmed, which defeats the purpose of the review.
