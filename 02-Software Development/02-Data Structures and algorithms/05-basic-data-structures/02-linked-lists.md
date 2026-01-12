---
---

# Linked Lists

## 1. Abstract
A **Linked List** is a linear data structure where elements (nodes) are stored in non-contiguous memory locations. Each node contains two main components: the data itself and a reference (or pointer) to the next node in the sequence. Unlike arrays or slices, linked lists do not require a fixed size during initialization and allow for efficient **O(1)** insertion and deletion of elements at any position, provided a reference to the node is available. However, they suffer from **O(n)** access time as elements must be traversed sequentially.

## 2. Development

### Types of Linked Lists
1.  **Singly Linked List**: Each node points only to the next node. Navigation is forward-only.
    -   `Head -> [Data|Next] -> [Data|Next] -> nil`
2.  **Doubly Linked List**: Each node points to both the next and the previous node. Allows bidirectional traversal.
    -   `nil <- [Prev|Data|Next] <-> [Prev|Data|Next] -> nil`
3.  **Circular Linked List**: The last node points back to the head (singly) or the head's previous points to the tail (doubly), forming a circle.

### Time Complexity Comparison

| Operation | Array/Slice (Dynamic) | Singly Linked List | Doubly Linked List |
| :--- | :--- | :--- | :--- |
| **Access/Get** | O(1) | O(n) | O(n) |
| **Search** | O(n) | O(n) | O(n) |
| **Insert at Head** | O(n) (shift needed) | **O(1)** | **O(1)** |
| **Insert at Tail** | O(1) (amortized) | O(n) (or O(1) with tail ptr) | O(1) |
| **Insert Middle** | O(n) (shift needed) | O(1) (if ptr known) | O(1) (if ptr known) |
| **Delete** | O(n) (shift needed) | O(1) (if ptr known*) | O(1) (if ptr known) |

*> Note: For Singly Linked List, deleting a node O(1) requires the pointer to the **previous** node, or a trick of copying the next node's value and deleting the next node.*

### Memory Overhead
Linked Lists consume more memory than arrays due to the storage requirements for pointers (4 or 8 bytes per pointer on 32/64-bit systems).

## 3. Code Examples (Go)

### Idiomatic Singly Linked List
Go does not have a built-in singly linked list. Here is a standard implementation:

```go
package main

import "fmt"

// Node represents a single element in the list
type Node struct {
	Value int
	Next  *Node
}

// LinkedList is the wrapper for the head node
type LinkedList struct {
	Head *Node
	Size int
}

// AddFront inserts a new value at the beginning
func (l *LinkedList) AddFront(val int) {
	newNode := &Node{Value: val, Next: l.Head}
	l.Head = newNode
	l.Size++
}

// Traverse prints all values
func (l *LinkedList) Traverse() {
	current := l.Head
	for current != nil {
		fmt.Printf("%d -> ", current.Value)
		current = current.Next
	}
	fmt.Println("nil")
}

func main() {
	ll := &LinkedList{}
	ll.AddFront(3)
	ll.AddFront(2)
	ll.AddFront(1)
	ll.Traverse() // Output: 1 -> 2 -> 3 -> nil
}
```

### Standard Library: Doubly Linked List (`container/list`)
Go's standard library provides a doubly linked list implementation via `container/list`.

```go
package main

import (
	"container/list"
	"fmt"
)

func main() {
	// Initialize a new list
	l := list.New()

	// Push elements
	e4 := l.PushBack(4)
	e1 := l.PushFront(1)
	
	// Insert relative to other elements
	l.InsertBefore(3, e4)
	l.InsertAfter(2, e1)

	// Iterate
	// Note: 'Value' is of type interface{}, so type assertion might be needed
	for e := l.Front(); e != nil; e = e.Next() {
		fmt.Printf("%v ", e.Value)
	}
	// Output: 1 2 3 4
}
```

## 4. Go Application & Ecosystem

### The "Slice vs Linked List" Debate in Go
In Go, **Slices are almost always preferred over Linked Lists**.

1.  **Cache Locality**: Slices are backed by contiguous arrays. Modern CPUs rely heavily on caching. Traversing a slice pre-fetches data efficiently. Linked Lists scatter nodes across the heap, causing frequent cache misses.
2.  **Memory Layout**: A `[]int` stores integers packed together. A `*list.List` stores `*list.Element` structs which point to `interface{}` values, adding massive overhead (pointer + interface header + heap allocation per node).
3.  **Performance**: Benchmarks consistently show slices outperforming lists for iteration and even insertion (until the slice gets very large) due to the factors above.

### When to use Linked Lists in Go?
Despite the downsides, Linked Lists are used in specific scenarios:
-   **LRU Caches**: When you need to constantly move an accessed element to the "front" of the queue in O(1) time without shifting all other elements.
-   **Ring Buffers**: The `container/ring` package implements circular lists, useful for round-robin scheduling or fixed-size history buffers.
-   **Concurrent Data Structures**: Some lock-free data structures rely on atomic pointer swaps which are natural to linked lists.

## 5. Interview Preparation

### Common Questions

**Q: Why is a Linked List preferred over an Array for certain scenarios?**
> A: It allows dynamic memory allocation without reallocating/copying the entire structure (unlike a slice growing) and provides O(1) insertion/deletion if the pointer is known. It's efficient when the size is unpredictable and changes frequently.

**Q: How do you detect a cycle in a Linked List?**
> A: Use **Floyd’s Cycle-Finding Algorithm** (Tortoise and Hare). Initialize two pointers, slow (moves 1 step) and fast (moves 2 steps). If they meet, there is a cycle. If fast reaches `nil`, there is no cycle.

**Q: How do you find the middle element of a Singly Linked List in one pass?**
> A: Use two pointers. Move the `fast` pointer two steps and the `slow` pointer one step. When `fast` reaches the end, `slow` will be at the middle.

**Q: Reverse a Singly Linked List in-place.**
> A: Iterate through the list using three pointers: `prev`, `curr`, and `next`. For each node:
> 1. Save `next = curr.Next`
> 2. Reverse `curr.Next = prev`
> 3. Move `prev = curr`
> 4. Move `curr = next`
> Finally, set the Head to `prev`.
