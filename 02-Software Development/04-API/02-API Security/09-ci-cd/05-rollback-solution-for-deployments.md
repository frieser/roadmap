#API
---
---

## Summary
A rollback strategy is a planned procedure to revert an application to a previous stable state when a new deployment fails or introduces critical bugs. Effective rollback capabilities are essential for "Continuous Deployment" to ensure high availability and minimize Mean Time to Recovery (MTTR).

## Detailed Explanation

### Strategies
1.  **Blue/Green Deployment**: Two identical environments (Blue=Live, Green=New).
    *   *Deploy*: Update Green.
    *   *Switch*: Update Load Balancer to point to Green.
    *   *Rollback*: Point Load Balancer back to Blue (Instant).
2.  **Canary Deployment**: Roll out to a small % of users (e.g., 5%).
    *   *Rollback*: If errors spike, route traffic back to the 95% stable fleet.
3.  **Feature Flags**: Deploy code to 100% of servers, but hide it behind a flag.
    *   *Rollback*: Toggle the flag off (No re-deployment needed).
4.  **Re-deployment**: Build and deploy the *previous* git commit. (Slowest).

### Database Rollbacks
The hardest part. If the new version changed the DB schema (e.g., dropped a column), the old code might crash.
*   **Rule**: Changes must be **backward compatible**. (e.g., Add column -> Deploy Code -> Fill Column -> Remove Old Column).

## Go-Specific Context/Examples

Go applications compile to a single, static binary. This makes rollbacks incredibly simple compared to interpreted languages (Python/Node) where dependencies might drift.

### Container/Binary Swap
In a Kubernetes environment, rolling back a Go app is just changing the image tag.

```yaml
# Kubernetes Deployment
spec:
  containers:
  - name: my-go-app
    image: my-registry/app:v2.0.0  # ERROR found!
    # Rollback command: kubectl rollout undo deployment/my-go-app
    # Reverts image to: my-registry/app:v1.9.0
```

### Feature Flag Example in Go
Using a simple flag to disable a new feature without redeploying.

```go
if config.FeatureXEnabled {
    RunNewAlgorithm()
} else {
    RunStableAlgorithm()
}
```

## Interview Questions

**Q: Why are database migrations the bottleneck of rollbacks?**
**A:** Code is stateless and easy to version/swap. Data has state. You cannot easily "undo" a `DROP TABLE` or a data transformation that happened during the new version's runtime without restoring from a backup, which causes data loss. Backward-compatible migration strategies are required.

**Q: What is the difference between "Rolling Update" and "Blue/Green"?**
**A:** A Rolling Update replaces instances one by one (e.g., 10 servers, update 1 at a time). If an error occurs midway, you have a "mixed version" state. Blue/Green switches traffic all at once (or quickly), ensuring all users see one version, and allows for an instant "flip the switch" rollback.

**Q: What is MTTR and why does it matter?**
**A:** **Mean Time To Recovery**. It measures the average time it takes to fix a broken system. Fast rollback strategies (like Feature Flags or Blue/Green) drastically reduce MTTR (from hours to seconds), which is often more valuable than trying to prevent every possible bug (MTBF).
