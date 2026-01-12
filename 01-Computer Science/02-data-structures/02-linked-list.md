---
---

# Linked List

## Abstract
A **Linked List** is a linear data structure consisting of a sequence of elements called **nodes**. Unlike arrays, these elements are **not** stored in contiguous memory locations. Instead, each node contains data and a **reference** (or pointer) to the next node in the sequence. This structure allows for efficient insertion and deletion of elements from any position in the sequence, but at the cost of sequential access (no random access).

## Development

### Core Concept
The defining characteristic of a linked list is that the logical order of elements is determined by pointers, not physical memory placement.

**Node Structure**:
- **Data**: The value stored.
- **Next**: A pointer/reference to the next node.
- **Prev** (Optional): A pointer to the previous node (in Doubly Linked Lists).

**Visual Memory Layout**:
Nodes can be scattered anywhere in the heap.
```text
[Data|Next] -> [Data|Next] -> [Data|Next] -> NULL
  Addr: 500      Addr: 920      Addr: 310
```

### Common Types
1. **Singly Linked List**: Each node points only to the next node. Traversal is one-way.
2. **Doubly Linked List**: Each node has pointers to both **next** and **previous** nodes. Allows bidirectional traversal but requires more memory.
3. **Circular Linked List**: The last node points back to the first node instead of NULL, forming a circle.

### Time Complexity

| Operation | Complexity | Description |
|-----------|------------|-------------|
| **Access**| O(n)       | Must traverse from head to element `k`. |
| **Search**| O(n)       | Linear scan. |
| **Insert**| **O(1)**   | If pointer to position is known (e.g., insert at head). |
| **Delete**| **O(1)**   | If pointer to node (and prev node for Singly) is known. |

### Pros & Cons (vs Arrays)
- **Pros**: Dynamic size (no pre-allocation), easy insertion/deletion.
- **Cons**: High memory overhead (pointers take extra space), **poor cache locality** (CPU cache misses due to non-contiguous memory), no random access.

## Code Examples (Go)

In Go, you can implement a custom linked list or use the standard library.

### 1. Custom Singly Linked List (Generic)
A custom implementation is often preferred in Go to avoid `interface{}` overhead and ensure type safety.

```go
package main

import "fmt"

// Node represents a single element in the list
type Node[T any] struct {
	Value T
	Next  *Node[T]
}

// LinkedList is the wrapper structure
type LinkedList[T any] struct {
	Head *Node[T]
	Size int
}

// Append adds a value to the end
func (ll *LinkedList[T]) Append(value T) {
	newNode := &Node[T]{Value: value}
	if ll.Head == nil {
		ll.Head = newNode
		ll.Size++
		return
	}
	
	current := ll.Head
	for current.Next != nil {
		current = current.Next
	}
	current.Next = newNode
	ll.Size++
}

// Display prints the list
func (ll *LinkedList[T]) Display() {
	current := ll.Head
	for current != nil {
		fmt.Printf("%v -> ", current.Value)
		current = current.Next
	}
	fmt.Println("nil")
}

func main() {
	ll := LinkedList[int]{}
	ll.Append(10)
	ll.Append(20)
	ll.Append(30)
	
	ll.Display() // Output: 10 -> 20 -> 30 -> nil
}
```

### 2. Standard Library (`container/list`)
Go provides a **Doubly Linked List** implementation in the `container/list` package. Note that it uses `any` (interface{}), so type assertions are required.

```go
package main

import (
	"container/list"
	"fmt"
)

func main() {
	// Create a new doubly linked list
	l := list.New()
	
	// PushBack returns the Element, which can be used for relative insertions
	e4 := l.PushBack(4)
	e1 := l.PushFront(1)
	
	// Insert relative to other elements
	l.InsertBefore(3, e4) // List: 1, 3, 4
	l.InsertAfter(2, e1)  // List: 1, 2, 3, 4
	
	// Iterating
	for e := l.Front(); e != nil; e = e.Next() {
		// e.Value is 'any', so we treat it generally or assert type
		fmt.Printf("%v ", e.Value)
	}
}
```

## Go Application & Ecosystem

### The "Slice Preference"
In 99% of Go applications, **Slices** are superior to Linked Lists.
- **Cache Locality**: Slices (arrays) are contiguous, making them extremely friendly to CPU caches and prefetchers. Linked lists cause "pointer chasing" which stalls the CPU.
- **Memory Overhead**: A slice of `int`s stores just integers. A linked list of `int`s stores integers plus 64-bit pointers (8 bytes) for every element (16 bytes for doubly linked).
- **GC Pressure**: Each node in a linked list is a separate allocation, increasing Garbage Collection work.

### When to Use Linked Lists?
Despite the downsides, they are useful in specific scenarios:
1. **LRU Caches**: Combining a `map` (for lookup) and a `Doubly Linked List` (for ordering) allows O(1) updates to "recency" by moving nodes to the front.
2. **Undo/Redo Stacks**: Where state changes are frequent and intermediate states need to be removed or re-ordered cheaply.
3. **Queues/Deques**: If you implement a FIFO queue where you constantly add/remove from ends, a linked list avoids the "shifting" cost of arrays (though `ring buffer` slices are often still faster).

## Interview Preparation

### Common Questions

1. **How do you reverse a Singly Linked List?**
   - **Answer**: Iterate through the list with three pointers: `prev`, `current`, and `next`. For each node, save `next`, point `current.Next` to `prev`, then shift `prev` and `current` forward. Time: O(n), Space: O(1).

2. **How do you detect a cycle in a Linked List?**
   - **Answer**: Use **Floyd’s Cycle-Finding Algorithm** (Tortoise and Hare). Use two pointers: `slow` (moves 1 step) and `fast` (moves 2 steps). If they meet, there is a cycle. If `fast` reaches NULL, there is no cycle.

3. **What is the difference between `container/list` and a slice in Go?**
   - **Answer**: `container/list` is a doubly linked list using `interface{}`, offering O(1) insertions/deletions at known positions but poor cache performance. Slices are dynamic arrays with O(1) access and great cache locality but O(n) insertions/deletions (due to shifting).

4. **Find the middle of a Linked List in one pass.**
   - **Answer**: Use the two-pointer technique. Move `fast` pointer 2 steps and `slow` pointer 1 step. When `fast` reaches the end, `slow` will be at the middle.

5. **Merge two sorted Linked Lists.**
   - **Answer**: Create a dummy head node. Compare the heads of both lists, attach the smaller one to the dummy's current pointer, and advance. Repeat until one list is empty, then attach the remainder of the other list.
