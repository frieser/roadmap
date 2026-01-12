## Summary
Mentoring programs are formal structures to pair experienced engineers with those looking to grow. They are a pillar of retention and skill development. A good program has clear goals, time limits, and matching criteria.

## Detailed Explanation
Mentorship is distinct from management. A mentor is a guide, not a boss.

### Types
*   **Career Mentoring**: Focus on soft skills, promotion paths, and office politics.
*   **Technical Mentoring**: Focus on Go patterns, system design, and code quality.
*   **Onboarding Buddy**: Short-term tactical help for new hires.

## Go Code Example
Modeling a `MentorMatcher` that pairs users based on desired skills and available expertise.

```go
package main

import "fmt"

type Profile struct {
	Name        string
	Expertise   []string
	WantsToLearn []string
}

func FindMatch(mentor, mentee Profile) bool {
	for _, want := range mentee.WantsToLearn {
		for _, have := range mentor.Expertise {
			if want == have {
				return true
			}
		}
	}
	return false
}

func main() {
	senior := Profile{Name: "Alice", Expertise: []string{"Go", "K8s", "Architecture"}}
	junior := Profile{Name: "Bob", WantsToLearn: []string{"Go", "Rust"}}

	if FindMatch(senior, junior) {
		fmt.Printf("Match found! %s can mentor %s in Go.\n", senior.Name, junior.Name)
	}
}
```

## Interview Questions
**Q: What is the difference between a mentor and a manager?**
**A:** A manager controls your salary, promotion, and project assignments. A mentor is a safe space to discuss gaps and fears without performance anxiety. A manager *can* mentor, but a separate mentor is often better for psychological safety.

**Q: How do you measure the success of a mentoring program?**
**A:** Qualitative feedback surveys (NPS), promotion rates of mentees vs. non-mentees, and retention rates of both mentors (feeling valued) and mentees (feeling supported).
