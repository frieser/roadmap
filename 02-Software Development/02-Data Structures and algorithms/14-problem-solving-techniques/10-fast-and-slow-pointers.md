---
---

# Fast and Slow Pointers

## Summary
**Fast and Slow Pointers** (Tortoise and Hare algorithm) involves using two pointers moving at different speeds (usually 1 step vs 2 steps). It is famously used for **Cycle Detection** in Linked Lists and Arrays.

## Detailed Explanation

### Mechanism
*   **Slow**: Moves 1 step at a time.
*   **Fast**: Moves 2 steps at a time.
*   **Collision**: If there is a cycle, Fast will eventually lap Slow and they will meet inside the cycle.
*   **No Cycle**: Fast will reach the end (`nil`) first.

## Code Examples (Go)

### Cycle Detection in Linked List
```go
func HasCycle(head *ListNode) bool {
    slow, fast := head, head
    
    for fast != nil && fast.Next != nil {
        slow = slow.Next
        fast = fast.Next.Next
        
        if slow == fast {
            return true
        }
    }
    return false
}
```
