---
---

# Huffman Coding

## Abstract
**Huffman Coding** is a popular greedy algorithm used for **lossless data compression**. It assigns variable-length binary codes to characters based on their frequency. More frequent characters get shorter codes, and less frequent characters get longer codes. It guarantees the **optimal prefix-free code**.

## Development

### Core Concept
1.  **Frequency Count**: Count occurences of all characters.
2.  **Priority Queue**: Create a leaf node for each char and push to Min-Heap (ordered by frequency).
3.  **Build Tree**:
    -   Pop two smallest nodes ($min1, min2$).
    -   Create a new internal node with `frequency = min1.freq + min2.freq`.
    -   Set $min1$ as left child, $min2$ as right.
    -   Push new node back to Heap.
    -   Repeat until 1 node remains (Root).
4.  **Assign Codes**: Traverse tree (Left=0, Right=1) to generate codes.

### Complexity
-   **Time**: $O(n \log n)$ where $n$ is number of unique characters (due to heap operations).
-   **Space**: $O(n)$ to store the tree.

## Code Examples (Go)

```go
package main

import (
    "container/heap"
    "fmt"
)

type Node struct {
    Char  rune
    Freq  int
    Left  *Node
    Right *Node
}

// PriorityQueue boilerplate omitted for brevity (sort by Freq)

func BuildHuffmanTree(text string) *Node {
    freqs := make(map[rune]int)
    for _, c := range text {
        freqs[c]++
    }

    pq := &PriorityQueue{}
    heap.Init(pq)
    for char, freq := range freqs {
        heap.Push(pq, &Node{Char: char, Freq: freq})
    }

    // Greedy Step: Merge two smallest
    for pq.Len() > 1 {
        left := heap.Pop(pq).(*Node)
        right := heap.Pop(pq).(*Node)
        merged := &Node{
            Freq:  left.Freq + right.Freq,
            Left:  left,
            Right: right,
        }
        heap.Push(pq, merged)
    }
    return heap.Pop(pq).(*Node)
}
```

## Go Application
-   **Compression**: Used in DEFLATE (ZIP, GZIP, PNG). Go's `compress/flate` implements Huffman coding.
-   **Encoding**: Optimizing bandwidth for known protocols.

## Interview Preparation
1.  **Prefix Property**: No code is a prefix of another (e.g., `a=0`, `b=01` is invalid because `0` is prefix of `01`). Huffman guarantees this by putting characters only at leaves.
2.  **Greedy Choice**: Why merge smallest? Merging them early pushes them deeper in the tree, giving them longer codes, which minimizes total weighted path length.
