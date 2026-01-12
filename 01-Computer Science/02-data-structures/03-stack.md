---
---

# Stack

## Abstract
A **Stack** is a linear data structure that follows the **LIFO** (Last In, First Out) principle. The last element added to the stack is the first one to be removed. It behaves exactly like a physical stack of plates: you can only add a new plate to the top or remove the top plate. This strict ordering makes it fundamental for managing function calls, expression evaluation, and backtracking algorithms.

## Development

### Core Concept
A stack is defined by two primary operations:
1.  **Push**: Add an element to the top.
2.  **Pop**: Remove the top element.

Additionally, **Peek** (or Top) allows viewing the top element without removing it.

**Visual Representation**:
```text
      |     |
      | [C] | <- Top (Last In)
      | [B] |
      | [A] | <- Bottom (First In)
      +-----+
```

### Time Complexity

| Operation | Complexity | Description |
|-----------|------------|-------------|
| **Push**  | **O(1)**   | Adding to the end of a dynamic array (amortized) or linked list. |
| **Pop**   | **O(1)**   | Removing from the end. |
| **Peek**  | **O(1)**   | Reading the last element. |
| **Search**| O(n)       | Linear scan to find an element. |

### Common Implementations
1.  **Array-based**: Uses a dynamic array (Slice in Go). Fast, cache-friendly, but resizing can cause occasional latency.
2.  **Linked List-based**: Uses a linked list. Constant time operations always, but higher memory overhead per element.

## Code Examples (Go)

In Go, there is no built-in `Stack` type. The **idiomatic** way is to use a **Slice**.

### 1. Slice-based Stack (Idiomatic)
This is the most common approach. It leverages the built-in `append` function and slice indexing.

```go
package main

import "fmt"

func main() {
	// 1. Create
	var stack []int

	// 2. Push (Append)
	stack = append(stack, 10)
	stack = append(stack, 20)
	stack = append(stack, 30)

	// 3. Peek (Top)
	if len(stack) > 0 {
		top := stack[len(stack)-1]
		fmt.Println("Top:", top) // 30
	}

	// 4. Pop (Remove last)
	if len(stack) > 0 {
		// Get value
		popVal := stack[len(stack)-1]
		// Shrink slice
		stack = stack[:len(stack)-1] 
		fmt.Println("Popped:", popVal) // 30
	}
	
	fmt.Println("Stack:", stack) // [10 20]
}
```

### 2. Thread-Safe Generic Stack
For production systems requiring concurrent access or strict type boundaries, a wrapper struct with a Mutex is better.

```go
package main

import (
	"fmt"
	"sync"
)

// Stack is a thread-safe generic stack
type Stack[T any] struct {
	mu   sync.Mutex
	data []T
}

func (s *Stack[T]) Push(val T) {
	s.mu.Lock()
	defer s.mu.Unlock()
	s.data = append(s.data, val)
}

func (s *Stack[T]) Pop() (T, bool) {
	s.mu.Lock()
	defer s.mu.Unlock()

	if len(s.data) == 0 {
		var zero T
		return zero, false
	}

	index := len(s.data) - 1
	val := s.data[index]
	
	// Optional: Zero out element to avoid memory leak if T is a pointer
	// s.data[index] = *new(T)
	
	s.data = s.data[:index]
	return val, true
}

func main() {
	tsStack := &Stack[string]{}
	tsStack.Push("A")
	tsStack.Push("B")
	
	if val, ok := tsStack.Pop(); ok {
		fmt.Println("Popped:", val) // B
	}
}
```

## Go Application & Ecosystem

### The "Memory Leak" Gotcha
When using a Slice as a Stack for **pointers** or **structs with pointers**, simply slicing it `s[:len(s)-1]` is not enough. The underlying array **still holds a reference** to the popped element, preventing the Garbage Collector from freeing it.

**Correct Pop for Pointers**:
```go
// Assuming stack is []*MyStruct
index := len(stack) - 1
item := stack[index]
stack[index] = nil // <--- CRITICAL: Remove reference
stack = stack[:index]
```

### Use Cases
1.  **Function Call Stack**: The runtime uses a stack to keep track of active subroutines.
2.  **DFS (Depth-First Search)**: Iterative implementations of graph/tree traversals use a stack.
3.  **Undo/Redo**: Browsers and text editors use stacks to track state history.
4.  **Syntax Parsing**: Compilers use stacks to check for balanced parentheses and evaluate expressions (RPN).

## Interview Preparation

### Common Questions

1.  **Implement a Min-Stack.**
    *   **Question**: Design a stack that supports `push`, `pop`, `top`, and `getMin` in O(1) time.
    *   **Answer**: Maintain **two stacks**. One main stack for data, and a second `minStack` that only pushes the new value if it is smaller than or equal to the current minimum. When popping from main, if the value equals `minStack.top`, pop from `minStack` too.

2.  **Valid Parentheses.**
    *   **Question**: Given a string containing `()`, `{}`, `[]`, determine if it's valid.
    *   **Answer**: Iterate through the string. Push opening brackets. When a closing bracket appears, check if it matches the `stack.top`. If stack is empty or mismatch, return false. Finally, return `stack.isEmpty`.

3.  **Queue using Stacks.**
    *   **Question**: Implement a Queue (FIFO) using only Stacks (LIFO).
    *   **Answer**: Use two stacks: `Input` and `Output`. Push to `Input`. To pop/peek, check `Output`. If `Output` is empty, pop all elements from `Input` and push them to `Output` (reversing the order). Then pop from `Output`. Amortized O(1).

4.  **Why is a slice preferred over `container/list` for a stack in Go?**
    *   **Answer**: Performance. Slices are contiguous in memory (cache locality) and reduce allocations. `container/list` requires a separate heap allocation for every node (`Element`), causing significant GC pressure and cache misses.
