---
---

## Summary
The **Activity Diagram** describes the **dynamic behavior** of a system by modeling the flow of control from activity to activity. It is similar to a flowchart but with support for concurrency (parallel processing). It shows *how* an operation is executed, focusing on the sequence of steps.

## Detailed Explanation

Activity diagrams are used to model workflows, business processes, or the internal logic of a complex algorithm. They are excellent for visualizing parallel processing.

### Key Components

1.  **Start Node**: Solid black circle.
2.  **Activity (Action)**: Rounded rectangle representing a step.
3.  **Control Flow**: Arrow showing direction.
4.  **Decision Node**: Diamond shape for conditional branching (if/else).
5.  **Merge Node**: Diamond shape for merging branches back.
6.  **Fork Node**: Black bar splitting one flow into multiple parallel flows.
7.  **Join Node**: Black bar waiting for parallel flows to complete before proceeding.
8.  **End Node**: Solid circle with a border.

### Usage
*   **Business Process Modeling**: e.g., "Order Processing" (Check Stock -> (Ship + Bill) -> Close).
*   **Algorithm Visualization**: Mapping complex logic.
*   **Multithreaded Programming**: Designing concurrent workflows.

### Mermaid Example
```mermaid
activityDiagram
    start
    :Receive Order;
    if (Is Stock Available?) then (yes)
        :Process Payment;
        fork
            :Ship Item;
        fork again
            :Send Email;
        end fork
        :Close Order;
    else (no)
        :Cancel Order;
    endif
    stop
```

## Go Example

Activity diagrams map well to procedural code, especially with Go's **concurrency primitives** (`go routines` and `wait groups`) representing Forks and Joins.

```go
package main

import (
	"fmt"
	"sync"
)

// Represents the Activity Diagram for "Process Order"
func ProcessOrder(orderID int) {
	fmt.Println("(Start) Processing Order:", orderID)

	// Step 1: Check Stock (Activity)
	if !checkStock() { // Decision Node
		fmt.Println("(End) Out of stock, cancelling.")
		return
	}

	// Step 2: Fork (Parallel Processing)
	// We need to Ship Item AND Send Email simultaneously
	var wg sync.WaitGroup
	wg.Add(2)

	go func() {
		defer wg.Done()
		shipItem() // Activity in branch 1
	}()

	go func() {
		defer wg.Done()
		sendEmail() // Activity in branch 2
	}()

	// Step 3: Join (Wait for both to finish)
	wg.Wait()

	// Step 4: Finalize
	fmt.Println("(End) Order Closed.")
}

func checkStock() bool {
	return true
}

func shipItem() {
	fmt.Println(" -> Shipping Item...")
}

func sendEmail() {
	fmt.Println(" -> Sending Email...")
}

func main() {
	ProcessOrder(101)
}
```

## Interview Questions

### Q: What is the main difference between an Activity Diagram and a Flowchart?
**A:** While they look similar, Activity Diagrams support **concurrency** (fork/join) natively, whereas standard flowcharts are typically sequential. Activity diagrams are also formally defined in the UML standard with object-oriented semantics in mind.

### Q: How do you model a "Loop" in an Activity Diagram?
**A:** A loop is modeled by a control flow arrow going back to a previous activity or decision node, creating a cycle.

### Q: Activity Diagram vs. Sequence Diagram?
**A:** Activity diagrams focus on the **flow of control** (what happens next). Sequence diagrams focus on the **flow of messages** between objects (who talks to whom). Use Activity for algorithms; use Sequence for interactions.
