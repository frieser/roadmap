---
---

# Breadth First Search (BFS) / Level Order

## Abstract
**Breadth First Search (BFS)** for trees is often called **Level Order Traversal**. It processes nodes level by level, from left to right. Unlike depth-first traversals (Pre/In/Post) which use a stack, BFS uses a **Queue**. It is used to find the shortest path in unweighted graphs or to print a tree structure visually.

## Development

### Core Concept
1.  Enqueue Root.
2.  While Queue is not empty:
    -   Dequeue node.
    -   Process node.
    -   Enqueue Left child (if exists).
    -   Enqueue Right child (if exists).

### Difference from Graph BFS
In trees, there are **no cycles**, so we do **not** need a `visited` set to keep track of processed nodes. This simplifies the implementation and saves space.

### Complexity
- **Time**: $O(n)$.
- **Space**: $O(w)$ where $w$ is the maximum width of the tree. In a perfect binary tree, $w \approx n/2$, so space is $O(n)$.

## Code Examples (Go)

```go
func LevelOrder(root *TreeNode) [][]int {
    if root == nil {
        return nil
    }
    
    var result [][]int
    queue := []*TreeNode{root}
    
    for len(queue) > 0 {
        levelSize := len(queue)
        var currentLevel []int
        
        // Process all nodes at current level
        for i := 0; i < levelSize; i++ {
            node := queue[0]
            queue = queue[1:] // Dequeue
            
            currentLevel = append(currentLevel, node.Val)
            
            if node.Left != nil {
                queue = append(queue, node.Left)
            }
            if node.Right != nil {
                queue = append(queue, node.Right)
            }
        }
        result = append(result, currentLevel)
    }
    return result
}
```

## Go Application
- **Tree Serialization**: Level order serialization (like LeetCode's input format `[1,2,3,null,null,4,5]`) is compact and common.
- **Shortest Path**: Finding the nearest leaf node to the root.

## Interview Preparation
1.  **ZigZag Traversal**: A common variation where levels are printed left-to-right, then right-to-left.
2.  **Right Side View**: Return the last element of `currentLevel` for each iteration.
