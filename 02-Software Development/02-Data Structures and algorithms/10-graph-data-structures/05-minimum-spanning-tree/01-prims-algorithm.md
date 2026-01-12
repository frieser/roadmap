---
---

# Prim's Algorithm

## Summary
**Prim's Algorithm** is a greedy algorithm that finds a **Minimum Spanning Tree (MST)** for a weighted undirected graph. It starts from an arbitrary node and grows the tree one edge at a time by choosing the cheapest edge connecting a tree node to a non-tree node.

## Detailed Explanation

### Mechanism
1.  Maintain two sets: `MSTSet` (included vertices) and `NonMSTSet`.
2.  Use a **Priority Queue** to store edges connected to the `MSTSet`.
3.  Start with any node.
4.  Repeat until all vertices are in `MSTSet`:
    *   Pick the minimum weight edge $(u, v)$ where $u \in MST$ and $v \notin MST$.
    *   Add $v$ to `MSTSet`.
    *   Add all edges connected to $v$ to the PQ.

### Complexity
*   **Time**: $O(E \log V)$ with Binary Heap.
*   **Space**: $O(V + E)$.

## Use Cases
1.  **Network Design**: Laying cables to connect cities with min cost.
2.  **Circuit Design**: Connecting pins on a chip.

## Interview Questions

**Q: Prim's vs Kruskal's?**
**A:**
*   **Prim's**: Better for **Dense Graphs** (lots of edges).
*   **Kruskal's**: Better for **Sparse Graphs** (fewer edges).
