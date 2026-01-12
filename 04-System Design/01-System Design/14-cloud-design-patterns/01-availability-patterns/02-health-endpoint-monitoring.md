---
---

## Summary
**Health Endpoint Monitoring** is a pattern where a service exposes a specific API endpoint (e.g., `/health` or `/status`) that external tools (like load balancers or orchestrators) can check to verify the service is running and capable of handling requests. It enables automated health checks, self-healing, and traffic routing decisions.

## Detailed Explanation
In cloud applications, services can fail in partial ways—the process might be running, but it can't connect to the database, or it's deadlocked. A simple "is the process alive" check is insufficient.

The Health Endpoint Monitoring pattern addresses this by having the application run internal functional checks and expose the aggregate result via an HTTP endpoint.

### Types of Checks
*   **Liveness**: Indicates if the application process is running. If this fails, the orchestrator (e.g., Kubernetes) kills and restarts the instance.
*   **Readiness**: Indicates if the application is ready to accept traffic. If this fails, the load balancer removes the instance from the pool until it passes again (e.g., waiting for cache to warm up or database connection).

### Implementation Best Practices
*   **Deep vs. Shallow**: A "shallow" check just returns 200 OK. A "deep" check verifies dependencies (DB, Cache, Storage). Be careful with deep checks; if the health check itself is heavy, frequent polling can degrade performance.
*   **Security**: Health endpoints can leak system info. They should be secured or exposed on a private internal port/network.

### Go Implementation
A standard Go implementation using `net/http` to expose a health endpoint that checks a database connection.

```go
package main

import (
	"context"
	"database/sql"
	"encoding/json"
	"net/http"
	"time"
)

// HealthResponse represents the structure of the health check output
type HealthResponse struct {
	Status    string            `json:"status"`
	Timestamp time.Time         `json:"timestamp"`
	Checks    map[string]string `json:"checks"`
}

func HealthHandler(db *sql.DB) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		// Set a timeout to prevent the health check from hanging
		ctx, cancel := context.WithTimeout(r.Context(), 2*time.Second)
		defer cancel()

		status := "UP"
		checks := make(map[string]string)

		// 1. Check Database Connectivity
		if err := db.PingContext(ctx); err != nil {
			status = "DOWN"
			checks["database"] = "unreachable: " + err.Error()
		} else {
			checks["database"] = "ok"
		}

		// 2. (Optional) Check other dependencies like Redis, Disk Space, etc.

		resp := HealthResponse{
			Status:    status,
			Timestamp: time.Now(),
			Checks:    checks,
		}

		w.Header().Set("Content-Type", "application/json")
		// If any check failed, return a 503 Service Unavailable
		if status == "DOWN" {
			w.WriteHeader(http.StatusServiceUnavailable)
		}
		
		json.NewEncoder(w).Encode(resp)
	}
}

/* 
Usage in main:
func main() {
    db := initDB()
    http.HandleFunc("/healthz", HealthHandler(db))
    http.ListenAndServe(":8080", nil)
}
*/
```

## Interview Questions

**Q: What is the difference between Liveness and Readiness probes in Kubernetes?**
**A:** 
*   **Liveness Probe**: Checks if the application is alive. If it fails, Kubernetes **restarts** the container (assumes the app is deadlocked or crashed).
*   **Readiness Probe**: Checks if the application is ready to serve traffic. If it fails, Kubernetes **stops sending traffic** to it (removes it from endpoints) but does not restart it. This is useful during startup or temporary overload.

**Q: Should a health check verify all downstream dependencies?**
**A:** It depends. For a **Readiness** probe, generally yes—if the app can't reach its DB, it can't serve traffic. However, for a **Liveness** probe, checking external dependencies is risky. If the DB goes down, you don't want *all* your app instances to restart simultaneously (cascading failure). Liveness checks should usually be local (e.g., "is the HTTP server loop running?").

**Q: How do you secure a health endpoint?**
**A:** Since health endpoints can reveal infrastructure details (e.g., "Redis is down"), they should be protected. Common strategies include:
1.  Running the health server on a separate, non-public port (e.g., admin port 9090).
2.  Restricting access to internal IP ranges (e.g., only the Load Balancer's subnet).
3.  Requiring a specific HTTP header or API key (though this complicates the load balancer configuration).
