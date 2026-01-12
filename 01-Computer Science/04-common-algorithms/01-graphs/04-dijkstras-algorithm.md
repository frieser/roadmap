---
---

# Dijkstra's Algorithm

## Abstract
**Dijkstra's Algorithm** finds the shortest paths from a source node to all other nodes in a graph with **non-negative** edge weights. It is the gold standard for routing and pathfinding. It uses a **greedy approach** and a **Priority Queue** to explore the most promising (shortest distance) nodes first.

## Development

### Core Concept
1.  Assign tentative distance 0 to source, $\infty$ to others.
2.  Add source to a Min-Priority Queue.
3.  While PQ is not empty:
    -   Extract node $u$ with smallest distance.
    -   For each neighbor $v$ of $u$ with weight $w$:
        -   Relax: If $dist[u] + w < dist[v]$:
            -   $dist[v] = dist[u] + w$.
            -   Add/Update $v$ in PQ.

### Complexity
- **Time**: $O(E \log V)$ using a Binary Heap.
- **Space**: $O(V)$.

## Code Examples (Go)

Go requires implementing `heap.Interface` to use a Priority Queue.

```go
package main

import (
    "container/heap"
    "fmt"
    "math"
)

// Item for Priority Queue
type Item struct {
    Node     int
    Priority float64 // Distance
    index    int
}

// PriorityQueue implementation
type PriorityQueue []*Item

func (pq PriorityQueue) Len() int { return len(pq) }
func (pq PriorityQueue) Less(i, j int) bool {
    return pq[i].Priority < pq[j].Priority
}
func (pq PriorityQueue) Swap(i, j int) {
    pq[i], pq[j] = pq[j], pq[i]
    pq[i].index = i
    pq[j].index = j
}
func (pq *PriorityQueue) Push(x any) {
    n := len(*pq)
    item := x.(*Item)
    item.index = n
    *pq = append(*pq, item)
}
func (pq *PriorityQueue) Pop() any {
    old := *pq
    n := len(old)
    item := old[n-1]
    item.index = -1
    *pq = old[0 : n-1]
    return item
}

// Dijkstra Algorithm
func Dijkstra(graph map[int]map[int]float64, start int) map[int]float64 {
    dist := make(map[int]float64)
    for node := range graph {
        dist[node] = math.Inf(1)
    }
    dist[start] = 0

    pq := &PriorityQueue{}
    heap.Init(pq)
    heap.Push(pq, &Item{Node: start, Priority: 0})

    for pq.Len() > 0 {
        u := heap.Pop(pq).(*Item)

        // Optimization: Skip if we found a shorter path already
        if u.Priority > dist[u.Node] {
            continue
        }

        for v, weight := range graph[u.Node] {
            if dist[u.Node]+weight < dist[v] {
                dist[v] = dist[u.Node] + weight
                heap.Push(pq, &Item{Node: v, Priority: dist[v]})
            }
        }
    }
    return dist
}
```

## Go Application
- **Maps Services**: Calculating driving directions.
- **Network Routing**: OSPF (Open Shortest Path First) protocol.
- **Microservices**: Finding optimal request path in a service mesh (less common, usually load balanced).

## Interview Preparation
1.  **Why non-negative weights?**
    -   *Answer*: Dijkstra assumes that "visiting a node" finalizes its shortest path. A negative edge could later reveal a shorter path back to an already visited node, breaking the greedy assumption.
2.  **Complexity without Heap?**
    -   *Answer*: $O(V^2)$ if using a simple array search for the minimum distance node. Better for dense graphs ($E \approx V^2$), worse for sparse ones.
