---
---

## Summary
The "People" pillar is the foundation of Engineering Management, shifting the focus from code to the humans who write it. Its primary goal is to build, grow, and sustain a high-performing, healthy team. An EM succeeds here by mastering hiring, mentoring, performance management, and career development, ensuring that every engineer has the support they need to thrive.

## Detailed Explanation

### 1. The Core Responsibilities
The People pillar can be broken down into four main activities:

#### A. Hiring and Recruitment
*   **Pipeline Management**: Partnering with recruiters to define roles and source candidates.
*   **Interviewing**: Designing structured loops that minimize bias and assess true signal.
*   **Closing**: Selling the vision and culture to top talent.

#### B. Mentoring and Coaching
*   **Mentoring**: Sharing your experience ("Here is how I did it").
*   **Coaching**: Helping them find their own answers ("What do you think is the best path?").
*   **Sponsorship**: Using your political capital to advocate for them when they aren't in the room.

#### C. Performance Management
*   **Feedback Loops**: Continuous, actionable feedback (both praise and constructive).
*   **Reviews**: Formal cycles to document achievements and areas for growth.
*   **Underperformance**: Managing PIPs (Performance Improvement Plans) compassionately but firmly when necessary.

#### D. Psychological Safety
*   Creating an environment where engineers feel safe to take risks, admit mistakes, and disagree without fear of retribution. This is the #1 predictor of high-performing teams (Google's Project Aristotle).

### 2. The Shift from IC to Manager
For a new EM, the People pillar is often the most challenging shift.
*   **Relationships Change**: You are no longer a peer; you have power over salaries and careers.
*   **Emotional Labor**: You absorb the team's stress, anxiety, and interpersonal conflicts.
*   **Delayed Gratification**: "Fixing" a person or a team dynamic takes months, unlike fixing a bug.

## Go Code Example: Structuring a Career Ladder
We can model the "People" growth path using Go structs to define levels and expectations. This helps visualize how expectations grow with seniority.

```go
package main

import "fmt"

// Level represents an engineering level
type Level struct {
	Title        string
	Expectations []string
	NextLevel    *Level
}

// Engineer represents a team member
type Engineer struct {
	Name    string
	Current *Level
}

// Promote attempts to move the engineer to the next level
func (e *Engineer) Promote() error {
	if e.Current.NextLevel == nil {
		return fmt.Errorf("engineer %s is already at the highest level", e.Name)
	}
	e.Current = e.Current.NextLevel
	fmt.Printf("Congratulations! %s has been promoted to %s\n", e.Name, e.Current.Title)
	return nil
}

func main() {
	// Define the ladder
	staff := &Level{Title: "Staff Engineer", Expectations: []string{"Set technical strategy", "Cross-team impact"}}
	senior := &Level{Title: "Senior Engineer", Expectations: []string{"Own complex systems", "Mentor juniors"}, NextLevel: staff}
	junior := &Level{Title: "Junior Engineer", Expectations: []string{"Learn codebase", "Ship features"}, NextLevel: senior}

	// New hire
	alice := Engineer{Name: "Alice", Current: junior}
	
	// Career growth journey
	alice.Promote() // Junior -> Senior
	alice.Promote() // Senior -> Staff
}
```

## Interview Questions

### Q: "How do you manage a high-performer with a toxic attitude?"
**A:** This is the "Brilliant Jerk" problem.
*   **Immediate Feedback**: Address the behavior immediately. "Your code is great, but how you spoke in the PR review hurt the team's psychological safety."
*   **Zero Tolerance**: Make it clear that cultural impact is part of performance.
*   **Outcome**: If they don't change, they must leave. A toxic high-performer destroys the output of the entire team.

### Q: "Tell me about a time you had to let someone go."
**A:** Focus on the process and dignity.
*   **No Surprises**: The individual should never be surprised by a firing. They should have had clear feedback and a chance to improve (PIP).
*   **Clarity**: Be clear about the reasons.
*   **Respect**: Treat them with dignity during the exit process.

### Q: "How do you handle 1:1s? What do you talk about?"
**A:** 1:1s are the employee's time, not a status update meeting.
*   **Topics**: Career growth, blockers, personal well-being, feedback.
*   **Frequency**: Weekly or bi-weekly.
*   **Rule**: Never cancel, only reschedule. Canceling signals they are not important.
