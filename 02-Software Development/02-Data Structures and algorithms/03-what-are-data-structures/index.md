---
---

# What are Data Structures?

## Summary
A **Data Structure** is a specialized format for organizing, processing, retrieving, and storing data in a computer. It defines not just the data itself, but the **relationships** between data items and the **operations** (like access, insertion, deletion) that can be performed on them. Choosing the right data structure is fundamental to writing efficient, scalable software, as it directly impacts an algorithm's Time and Space complexity ($O(n)$).

## Detailed Explanation

### 1. Definition and Purpose
In computer science, a data structure is a way of collecting and organizing data in such a way that we can perform operations on these data in an effective way.
*   **Data Organization**: How data is arranged in memory (e.g., contiguous vs. scattered).
*   **Data Management**: How data is accessed and modified.
*   **Efficiency**: Optimizing for speed (CPU) or memory (RAM).

### 2. Classification of Data Structures
Data structures are generally divided into two main categories:

#### A. Primitive vs. Non-Primitive
*   **Primitive**: Basic types provided directly by the language (e.g., `int`, `float`, `boolean`, `char`). They store single values.
*   **Non-Primitive**: Complex types derived from primitive types. They store multiple values or complex relationships.

#### B. Linear vs. Non-Linear (Non-Primitive)
This is the most common classification for structural design:

| Type | Description | Examples |
| :--- | :--- | :--- |
| **Linear** | Elements are arranged sequentially. Each element is connected to its previous and next element. | **Arrays**, **Linked Lists**, **Stacks** (LIFO), **Queues** (FIFO). |
| **Non-Linear** | Elements are arranged hierarchically. One element can be connected to multiple other elements. | **Trees** (BST, AVL), **Graphs**, **Heaps**, **Hash Tables** (conceptually). |

### 3. Static vs. Dynamic
*   **Static**: Size is fixed at compile time (e.g., traditional C arrays).
*   **Dynamic**: Size can grow or shrink at runtime (e.g., Linked Lists, Go Slices).

## Code Examples (Go)

Go is a strongly typed language that provides high-performance built-in data structures while allowing for custom implementations of theoretical structures.

### 1. Built-in Linear Structures (Arrays & Slices)
Go's `slice` is the most common linear data structure, providing a dynamic view over an underlying array.

```go
package main

import "fmt"

func main() {
    // 1. Array (Static, Fixed Size)
    var arr [5]int = [5]int{1, 2, 3, 4, 5}
    
    // 2. Slice (Dynamic, Growable)
    // Underlying structure: Pointer to array, Length, Capacity
    slice := []int{10, 20, 30}
    slice = append(slice, 40) // O(1) amortized insertion
    
    // 3. Map (Hash Table)
    // Key-Value store, Non-linear access
    users := make(map[string]int)
    users["alice"] = 1
    users["bob"] = 2
    
    fmt.Printf("Array: %v\nSlice: %v\nMap: %v\n", arr, slice, users)
}
```

### 2. Custom Structure (Structs)
To implement complex or non-linear structures (like a Linked List node), Go uses `structs` and pointers.

```go
package main

import "fmt"

// Node represents a data point in a generic Linked List
type Node[T any] struct {
    Value T
    Next  *Node[T]
}

func main() {
    // Creating a simple chain: A -> B -> nil
    nodeA := &Node[string]{Value: "Node A"}
    nodeB := &Node[string]{Value: "Node B"}
    
    nodeA.Next = nodeB
    
    fmt.Printf("Head: %s -> Next: %s\n", nodeA.Value, nodeA.Next.Value)
}
```

## Go Application
In the Go ecosystem, "traditional" CS data structures are handled differently:

1.  **Slices over Arrays**: Developers rarely use raw arrays. Slices provide the performance of contiguous memory (cache locality) with the convenience of dynamic sizing.
2.  **Maps**: The built-in `map` type is a highly optimized Hash Table.
3.  **`container` Package**: The standard library includes `container/list` (Doubly Linked List), `container/heap` (Heap interface), and `container/ring` (Circular list), though generic implementations are becoming more popular.
4.  **Composition**: Go encourages composition over inheritance. Complex data models are built by embedding structs within structs.

## Interview Questions

**Q: What is the difference between Linear and Non-Linear data structures?**
**A:** Linear structures arrange data sequentially (one after another), making them easy to traverse in a single run (e.g., Arrays, Stacks). Non-linear structures arrange data hierarchically or interconnectedly, allowing for complex relationships but requiring recursive or iterative traversal algorithms (e.g., Trees, Graphs).

**Q: Why are Arrays typically faster to iterate than Linked Lists?**
**A:** Arrays rely on **contiguous memory allocation**, which benefits from modern CPU caching (spatial locality). Linked Lists consist of scattered nodes connected by pointers; traversing them often causes "cache misses," forcing the CPU to fetch data from slower RAM more frequently.

**Q: How does Go implement a dynamic array?**
**A:** Go uses **Slices**. A slice is a lightweight descriptor containing a pointer to an underlying array, a length, and a capacity. When `append()` exceeds capacity, Go allocates a new, larger array (usually double the size), copies the existing elements, and updates the slice pointer.

**Q: Which data structure would you use to implement a recursive function without recursion?**
**A:** A **Stack**. Recursion relies on the Call Stack. Any recursive algorithm can be rewritten iteratively using an explicit Stack data structure to hold the state.
