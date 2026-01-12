---
---

## Summary
A robust **Development and Release Workflow** is the highway system for code. It defines how an idea travels from a developer's brain to the production environment. For an EM, the goal is to minimize friction and maximize safety. This involves choosing the right branching strategy (e.g., Trunk-Based Development), defining code review standards, and automating the release process to ensure that shipping software is a boring, predictable event.

## Detailed Explanation

### 1. Branching Strategies
*   **Trunk-Based Development**: Developers merge small, frequent updates to a core "main" branch. This is the gold standard for high-performing teams (DORA) as it reduces merge conflicts and encourages continuous integration.
*   **Gitflow**: A strict structure with `develop`, `release`, and `feature` branches. Good for legacy software with scheduled releases but often too slow for modern SaaS.
*   **GitHub Flow**: Simple feature branching. Create branch -> PR -> Merge to Main -> Deploy.

### 2. Code Review (Pull Requests)
*   **Purpose**: Quality control, knowledge sharing, and mentorship.
*   **Best Practices**:
    *   **Small Batches**: PRs should be small (< 400 lines) to ensure thorough review.
    *   **Speed**: Reviews should be picked up within hours, not days. Unreviewed code is inventory waste.
    *   **Tone**: Constructive, kind, and focused on the code, not the person.

### 3. Release Lifecycle
1.  **Code Freeze**: (Optional) Stopping new features to stabilize a release. Modern teams avoid this via automated testing.
2.  **Staging**: Deploying to a production-like environment for final verification.
3.  **Production**: The go-live moment.
4.  **Rollback Plan**: Every release must have an "Undo" button.

### 4. Hotfix Process
*   A predefined "emergency lane" for critical production bugs. It usually bypasses the standard queue but *must* still pass automated tests.

## Go Code Example: Workflow State Machine
We can model a Pull Request (PR) workflow using a Go struct and a state machine. This enforces the rules of the road (e.g., "You cannot merge without approval").

```go
package main

import (
	"errors"
	"fmt"
)

// State represents the status of a PR
type State string

const (
	Open      State = "OPEN"
	Reviewing State = "REVIEWING"
	Approved  State = "APPROVED"
	Merged    State = "MERGED"
	Closed    State = "CLOSED"
)

// PullRequest models a code change
type PullRequest struct {
	ID        int
	Author    string
	Status    State
	Approvals int
}

// Review allows a peer to approve the PR
func (pr *PullRequest) Review(approver string) error {
	if pr.Status == Merged || pr.Status == Closed {
		return errors.New("cannot review a closed PR")
	}
	if pr.Author == approver {
		return errors.New("authors cannot approve their own PR")
	}
	
	pr.Approvals++
	pr.Status = Reviewing
	
	if pr.Approvals >= 2 {
		pr.Status = Approved
		fmt.Printf("PR #%d is APPROVED by 2 reviewers.\n", pr.ID)
	}
	return nil
}

// Merge attempts to ship the code
func (pr *PullRequest) Merge() error {
	if pr.Status != Approved {
		return fmt.Errorf("PR #%d is not approved (Current: %s)", pr.ID, pr.Status)
	}
	
	// Simulate CI checks
	if !runCIChecks() {
		return errors.New("CI checks failed")
	}

	pr.Status = Merged
	fmt.Printf("PR #%d successfully MERGED to main.\n", pr.ID)
	return nil
}

func runCIChecks() bool {
	return true // Simulating passing tests
}

func main() {
	pr := PullRequest{ID: 101, Author: "Alice", Status: Open}
	
	// Attempt premature merge
	if err := pr.Merge(); err != nil {
		fmt.Println("Merge Failed:", err) // Expected failure
	}

	// Peer reviews
	pr.Review("Bob")
	pr.Review("Charlie")

	// Successful merge
	if err := pr.Merge(); err == nil {
		fmt.Println("Deployment started...")
	}
}
```

## Interview Questions

### Q: "Trunk-Based Development vs. Gitflow: Which do you prefer and why?"
**A:** I prefer Trunk-Based Development for modern SaaS applications.
*   **Why**: It forces smaller, incremental changes which reduces "merge hell." It aligns with Continuous Integration principles where main is always deployable.
*   **Context**: Gitflow is useful for versioned software (like boxed games or mobile apps) where you need to maintain multiple historical versions simultaneously.

### Q: "How do you speed up a slow Code Review culture?"
**A:** This is a workflow bottleneck.
*   **SLAs**: Set team expectations (e.g., "PRs must be reviewed within 24 hours").
*   **WIP Limits**: Stop starting new tickets if existing PRs are unreviewed. "Stop starting, start finishing."
*   **Pair Programming**: Skip the async review entirely by writing code together.

### Q: "Describe a robust hotfix process."
**A:**
1.  **Branch off Main/Tag**: Create the fix from the current production version.
2.  **Test**: Run the test suite. Do not skip tests just because it's urgent.
3.  **Deploy**: Ship to prod.
4.  **Cherry Pick**: Ensure the fix is immediately merged back into the development branch (main) so the bug doesn't regress in the next release.
