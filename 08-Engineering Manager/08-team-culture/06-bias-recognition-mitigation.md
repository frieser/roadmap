# Bias Recognition and Mitigation

## Summary
Bias Recognition and Mitigation is the practice of identifying unconscious mental shortcuts (heuristics) that lead to unfair or suboptimal decisions. Everyone has bias; it is a feature of the human brain, not a character flaw. The EM's job is to build *systems* that interrupt bias in hiring, promotion, and daily work.

## Detailed Explanation

### 1. Common Biases in Engineering
*   **Affinity Bias**: "I like him because he reminds me of myself" (went to same school, likes same games).
*   **Confirmation Bias**: Seeking evidence to confirm a first impression ("I thought he was smart, so I ignored his mistake").
*   **Attribution Bias**: "I failed because the docs were bad (External). He failed because he is incompetent (Internal)."
*   **Recency Bias**: Judging performance on the last 2 weeks.

### 2. Mitigation Strategies (System 2 Thinking)
*   **Slow Down**: Bias thrives in speed (System 1).
*   **Rubrics**: Define criteria *before* seeing the person.
*   **Blind Reviews**: Blind code screens, anonymized resumes.
*   **The "Flip It" Test**: "If a man did this, would I call him 'aggressive' or 'leader'?"

## Go Code Example: Bias Checker (Concept)
This example highlights how rubrics can mitigate scoring bias compared to "Gut Feeling."

```go
package main

import (
	"fmt"
)

type Candidate struct {
	Name       string
	School     string // Trigger for Affinity Bias
	TechScore  int
}

// BiasedEvaluation relies on "Gut Feeling"
func BiasedEvaluation(c Candidate, reviewerSchool string) int {
	score := c.TechScore
	if c.School == reviewerSchool {
		score += 2 // Affinity Bonus (Unconscious)
		fmt.Printf("[Bias Alert] Bumped score for %s (Alumni connection)\n", c.Name)
	}
	return score
}

// UnbiasedEvaluation relies on Rubric
func UnbiasedEvaluation(c Candidate) int {
	// Ignores School, focuses only on TechScore
	return c.TechScore
}

func main() {
	c1 := Candidate{"Alice", "MIT", 8}
	c2 := Candidate{"Bob", "State U", 8}
	
	reviewerSchool := "MIT"

	fmt.Println("--- Gut Feeling (Biased) ---")
	fmt.Printf("Alice Score: %d\n", BiasedEvaluation(c1, reviewerSchool))
	fmt.Printf("Bob Score:   %d\n", BiasedEvaluation(c2, reviewerSchool))

	fmt.Println("\n--- Structured Rubric (Unbiased) ---")
	fmt.Printf("Alice Score: %d\n", UnbiasedEvaluation(c1))
	fmt.Printf("Bob Score:   %d\n", UnbiasedEvaluation(c2))
}
```

## Interview Questions

### Q: "Tell me about a time you recognized your own bias."
**A:**
*   **Example**: "I caught myself assuming a candidate wasn't 'technical enough' because they were soft-spoken. I realized I was conflating extroversion with competence."
*   **Action**: "I looked strictly at their coding test results, which were excellent, and hired them."

### Q: "How do you mitigate bias in performance reviews?"
**A:**
*   **Calibration Meetings**: Managers compare ratings. "Why is Tom a 5 and Sarah a 3 when they shipped the same project?"
*   **Gender Decoder**: Check review text. Are we using "communal" words for women (helpful, supportive) and "agentic" words for men (driven, visionary)?

### Q: "What is 'Halo Effect' and how does it impact hiring?"
**A:**
*   **Definition**: One good trait (e.g., worked at Google) makes us assume everything else is good.
*   **Mitigation**: "Drill down." Verify skills independently. Don't assume competence by association.
