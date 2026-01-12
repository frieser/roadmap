---
---

## Summary
Pod Eviction is the process where the Kubernetes node (kubelet) terminates pods to reclaim resources when the node is under pressure (running out of Memory, Disk, or PID space). This safeguards the node stability.

## Detailed Explanation

### QoS Classes (Quality of Service)
Kubelet decides *who* to evict based on QoS:
1.  **Guaranteed** (Safest): Requests = Limits for CPU/Mem. Last to be evicted.
2.  **Burstable**: Requests < Limits. Evicted if using more than Request.
3.  **BestEffort** (Riskiest): No Requests/Limits. First to be evicted.

### Thresholds
Kubelet has default hard eviction thresholds:
*   `memory.available < 100Mi`
*   `nodefs.available < 10%`

### The OOMKiller
If memory runs out *instantly* (before kubelet can evict), the Linux Kernel OOMKiller steps in. It kills the process with the highest "OOM Score" (usually the one using the most memory relative to its request).

## Go-Specific Context/Examples

Go apps manage memory automatically (GC), but they treat all available RAM as usable. In K8s, you MUST set limits.

### Go GC Behavior
By default, Go's GC tries to use as much memory as possible to run faster. In K8s, this can hit the container limit and cause OOM.
*   **Mitigation**: Set `GOMEMLIMIT` (Go 1.19+) to roughly 90% of the Kubernetes memory limit. This tells the GC to be aggressive *before* the container gets killed.

## Interview Questions

**Q: What is the difference between Node-pressure Eviction and API-initiated Eviction?**
**A:**
*   **Node-pressure**: Kubelet kills pods because the *Node* is out of resources.
*   **API-initiated**: A user/controller requests eviction (e.g., `kubectl drain` or Pod Disruption Budget).

**Q: Why does a Pod get evicted with status "OOMKilled"?**
**A:** It means the container tried to use more RAM than its defined `resources.limits.memory`. The kernel killed it to protect the rest of the system.

**Q: How do you prevent your critical Go DB from being evicted?**
**A:** Set **Guaranteed** QoS (Requests == Limits) and a high **PriorityClass**.
