---
---

# Bellman-Ford Algorithm

## Summary
**Bellman-Ford** computes shortest paths from a source node to all other nodes, capable of handling **negative edge weights**. It can also **detect negative cycles** (situations where you can loop infinitely to reduce cost).

## Detailed Explanation

### Mechanism
1.  Initialize distances: Source = 0, others = $\infty$.
2.  **Relaxation**: Repeat $V-1$ times:
    *   For every edge $(u, v)$ with weight $w$:
        *   If `dist[u] + w < dist[v]`, update `dist[v]`.
3.  **Cycle Check**: Run one more time. If any distance changes, a negative cycle exists.

### Complexity
*   **Time**: $O(V \cdot E)$. (Slower than Dijkstra).
*   **Space**: $O(V)$.

## Go Application
Bellman-Ford is rarely used in standard web apps but is critical in **Network Routing Protocols** (like RIP) and finance (arbitrage detection).

## Interview Questions

**Q: Why do we relax edges $V-1$ times?**
**A:** The longest possible shortest path (without cycles) in a graph with $V$ vertices has $V-1$ edges. Each iteration propagates the shortest path information by at least one hop.

**Q: How do you detect negative cycles?**
**A:** If after $V-1$ iterations, you can *still* find a shorter path (`dist[u] + w < dist[v]`), it essentially means the path length can decrease infinitely, proving a negative cycle.
