---
---

# Breadth First Search (BFS)

## Summary
**Breadth First Search (BFS)** is a traversal algorithm that explores all nodes at the **current depth** level before moving on to nodes at the next depth level. It proceeds "layer by layer" (Level Order Traversal).

## Detailed Explanation

### Mechanism
BFS uses a **Queue** (FIFO) data structure.
1.  Add Root to Queue.
2.  While Queue is not empty:
    *   Dequeue a node.
    *   Process (Print) the node.
    *   Enqueue the node's Left Child (if exists).
    *   Enqueue the node's Right Child (if exists).

### Complexity
| Type | Complexity | Notes |
| :--- | :--- | :--- |
| **Time** | $O(n)$ | Visits every node once. |
| **Space** | $O(w)$ | Where $w$ is the max width of the tree. |

## Use Cases
1.  Finding the **Shortest Path** in an unweighted graph/tree.
2.  Printing a tree level-by-level.
3.  Web Crawlers (visit all links 1 hop away, then 2 hops away...).

## Code Examples (Go)

```go
package main

import "fmt"

type Node struct {
    Val   int
    Left  *Node
    Right *Node
}

func BFS(root *Node) {
    if root == nil {
        return
    }
    
    // Slice acts as a Queue
    queue := []*Node{root}
    
    for len(queue) > 0 {
        // Dequeue
        current := queue[0]
        queue = queue[1:]
        
        fmt.Printf("%d ", current.Val)
        
        // Enqueue children
        if current.Left != nil {
            queue = append(queue, current.Left)
        }
        if current.Right != nil {
            queue = append(queue, current.Right)
        }
    }
}
```
