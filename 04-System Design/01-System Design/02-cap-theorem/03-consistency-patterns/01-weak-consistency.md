---
---

## Summary
**Weak Consistency** is a consistency model where the system makes a "best effort" to propagate data, but offers no guarantee that a subsequent read will return the updated value. It prioritizes extreme speed and availability over data correctness. It is typically used for real-time data where occasional loss is acceptable.

## Detailed Explanation

### Core Characteristics
1.  **No Guarantees**: A read immediately after a write may return the old value, the new value, or no value at all.
2.  **Fire-and-Forget**: The writer sends data and doesn't wait for acknowledgment.
3.  **Speed**: Offers the lowest possible latency because there is no overhead for consensus or replication confirmation.

### Use Cases
*   **VoIP (Voice over IP) & Video Calls**: If a packet of audio/video is lost or delayed, it's better to skip it than to pause the call to retrieve it.
*   **Live Sports Stats**: A viewer seeing a score update 100ms later than another viewer is not critical.
*   **Gaming**: Player position updates in fast-paced shooters (clients predict movement, server corrects later).

## Go Example (Conceptual)
Using a buffered channel with a non-blocking select to simulate "best effort" delivery. If the receiver is slow, data is dropped.

```go
package main

import "fmt"

func main() {
	// Small buffer = Best effort queue
	stream := make(chan string, 1)

	sendUpdate := func(val string) {
		select {
		case stream <- val:
			fmt.Println("Sent:", val)
		default:
			// WEAK CONSISTENCY: Dropping data to maintain speed
			fmt.Println("Dropped (System Busy):", val)
		}
	}

	sendUpdate("Frame 1")
	sendUpdate("Frame 2") // Likely dropped if consumer hasn't read Frame 1 yet
}
```

## Interview Questions

### Q: When would you ever choose Weak Consistency?
**A:** When the value of the data decays rapidly over time. For example, in a live video stream, a frame that is 2 seconds old is useless. We prefer to drop it (weak consistency) and show the current frame rather than buffer and show old data (strong consistency).

### Q: How does it differ from Eventual Consistency?
**A:** Eventual Consistency guarantees that *if you stop writing*, all nodes will eventually converge to the correct value. Weak Consistency makes no such guarantee; data might be lost forever.
