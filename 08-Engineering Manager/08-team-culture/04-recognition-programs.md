# Recognition Programs

## Summary
Recognition Programs are systems for acknowledging and rewarding good work. Recognition is high-octane fuel for motivation. It reinforces desired behaviors (Values) and shows employees they are seen. It must be timely, specific, and calibrated to the individual's preference (public vs. private).

## Detailed Explanation

### 1. Types of Recognition
*   **Micro-Recognition**: Slack shoutouts, "Kudos" bot, verbal thanks in standup.
*   **Macro-Recognition**: Spot bonuses, "Employee of the Month," Promotion.
*   **Peer-to-Peer**: Often more meaningful than manager recognition because peers know the *real* difficulty of the work.

### 2. Public vs. Private
*   **Public**: Good for normalizing behavior ("Look, Alice fixed the build!").
*   **Private**: Good for introverts who hate the spotlight. *Always ask preference.*

### 3. The "Platinum Rule"
*   Golden Rule: Treat others as you want to be treated.
*   Platinum Rule: Treat others as *they* want to be treated. (Don't throw a surprise party for an introvert).

## Go Code Example: Kudos Budget
This example simulates a peer recognition system with a limited budget ("Spot Bonus").

```go
package main

import (
	"fmt"
)

type Employee struct {
	Name        string
	KudosPoints int
	Budget      int // Points they can give
}

func (e *Employee) GiveKudos(recipient *Employee, amount int, reason string) {
	if e.Budget >= amount {
		e.Budget -= amount
		recipient.KudosPoints += amount
		fmt.Printf("🏆 %s -> %s (+%d): %s\n", e.Name, recipient.Name, amount, reason)
	} else {
		fmt.Printf("❌ %s has insufficient budget!\n", e.Name)
	}
}

func main() {
	alice := &Employee{Name: "Alice", Budget: 100}
	bob := &Employee{Name: "Bob", Budget: 100}
	charlie := &Employee{Name: "Charlie", Budget: 100}

	// Peer Recognition Loop
	alice.GiveKudos(bob, 50, "Helped with the database migration")
	bob.GiveKudos(charlie, 20, "Fixed the broken build")
	charlie.GiveKudos(alice, 10, "Great code review feedback")

	fmt.Println("\n--- End of Month Totals ---")
	fmt.Printf("Alice: %d\n", alice.KudosPoints)
	fmt.Printf("Bob: %d\n", bob.KudosPoints)
	fmt.Printf("Charlie: %d\n", charlie.KudosPoints)
}
```

## Interview Questions

### Q: "How do you recognize an engineer who does 'invisible work' (e.g., refactoring, glue code)?"
**A:**
*   **Identify**: I dig into the commit logs and talk to the team. "Who helped you this week?"
*   **Amplify**: I explicitly highlight it in team meetings. "The build is 2x faster thanks to Jane's refactor."
*   **Incentivize**: Ensure it counts for promotion. If only "New Features" get promoted, the codebase rots.

### Q: "What if you have no budget for bonuses?"
**A:**
*   **Non-monetary works**:
    *   **Time**: "Take Friday off."
    *   **Access**: "Present your work to the CEO."
    *   **Growth**: "Pick the next conference to attend."
    *   **Sincere Thanks**: A handwritten note often means more than a $50 gift card.

### Q: "How do you ensure recognition is fair?"
**A:**
*   **Audit**: Check for bias. Are men getting credit for technical work and women for "glue work" (notes/organization)?
*   **Criteria**: Tie recognition to specific *Values* or *Results*, not just "being liked."
