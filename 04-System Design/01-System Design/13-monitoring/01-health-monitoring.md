---
---

# 01. Health Monitoring

## Summary
Health monitoring is the baseline for system reliability. It focuses on the binary question: "Is this component alive and capable of doing work?" In modern orchestration (Kubernetes), this is implemented through Liveness, Readiness, and Startup probes.

## Detailed Development

### 1. The Three Probes (Kubernetes Context)
Kubernetes uses different probes to manage the lifecycle of a container and ensure traffic is only sent to healthy instances.

| Probe Type | Mechanism | Action on Failure | Typical Use Case |
| :--- | :--- | :--- | :--- |
| **Liveness** | Checks if the process is stuck or deadlocked. | **Kills and restarts** the container. | Deadlocks, infinite loops. |
| **Readiness** | Checks if the app is ready to accept requests. | **Removes pod** from service endpoints (no traffic). | Loading large caches, database migrations. |
| **Startup** | Disables Liveness/Readiness until startup finishes. | Prevents premature restarts. | Legacy apps with long initialization. |

### 2. Probing Mechanisms
*   **HTTP GET**: Kubernetes sends a GET request to a specific path (e.g., `/healthz`). A response code `200 <= code < 400` is considered success.
*   **TCP Socket**: Kubernetes tries to open a TCP connection to the specified port.
*   **Exec Action**: Kubernetes executes a command inside the container (e.g., `pg_isready`). Exit code 0 is success.
*   **gRPC**: Introduced in K8s 1.24+, uses the standard gRPC Health Checking Protocol.

### 3. Best Practices
*   **Avoid External Dependencies in Liveness**: A Liveness probe should only check internal state. If your DB is down, don't fail the Liveness probe (restarting the app won't fix the DB), fail the **Readiness** probe instead.
*   **Separate Endpoints**: Use `/healthz` for liveness and `/readyz` for readiness.
*   **Graceful Shutdown**: Ensure your application handles `SIGTERM` and stops reporting "ready" before the process actually exits.

## Go Ecosystem
Go's standard library `net/http` makes implementing these probes trivial.

### Basic Probes in Go
**Evidence** ([source](https://github.com/kubernetes/ingress-gce/blob/master/cmd/echo/app/handlers.go#L93)):
```go
// Liveness: Process is alive
func healthHandler(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    w.Write([]byte("ok"))
}

// Readiness: App is ready for traffic
func readyHandler(w http.ResponseWriter, r *http.Request) {
    if !isDatabaseConnected() {
        http.Error(w, "database unreachable", http.StatusServiceUnavailable)
        return
    }
    w.WriteHeader(http.StatusOK)
    w.Write([]byte("ready"))
}
```

## Interview Preparation Questions

**Q1: What happens if a Liveness probe fails but a Readiness probe succeeds?**
**A1:** The container will still be restarted. Liveness takes precedence for the container lifecycle. However, this is a misconfiguration; if it's "live," it should generally be able to become "ready" eventually.

**Q2: Why should you avoid checking a database in a Liveness probe?**
**A2:** If the database goes down and 100 microservices fail their Liveness probes, Kubernetes will restart all 100 containers simultaneously. This creates a "thundering herd" effect and "restart loops" that can crash the entire cluster without fixing the root cause (the DB).

**Q3: How do you handle a "Zombie Process" that is alive but unresponsive?**
**A3:** Use a Liveness probe with a strict timeout. If the probe fails to receive an HTTP response within X seconds, K8s will terminate the zombie and start a fresh process.
