---
---

# 02. Availability Monitoring

## Summary
Availability monitoring measures the system's uptime from the perspective of the end-user (Black-box monitoring). While health checks tell you if the process is running, availability monitoring tells you if the service is actually usable across different networks and regions.

## Detailed Development

### 1. Uptime Checks (L1 Monitoring)
Simple, frequent probes to verify the service is reachable.
*   **ICMP (Ping)**: Basic network reachability. Rarely used for web apps due to firewalls.
*   **HTTP HEAD/GET**: Verifies the web server is responding. Usually checked from multiple global locations to detect regional ISP outages.
*   **DNS Monitoring**: Ensures the domain resolves to the correct IP addresses.

### 2. Synthetic Transactions (L2 Monitoring)
Simulates user behavior to detect functional failures that a simple HTTP 200 check would miss.
*   **Scripted Journeys**: A bot performs a sequence of actions (e.g., *Login -> Add Item -> Checkout*).
*   **Validation**: Asserts not just the status code, but the content of the page (e.g., "Welcome, User" must appear after login).
*   **Headless Browsers**: Uses Playwright or Selenium to execute JavaScript, catching frontend-only failures.

### 3. Service Level Indicators (SLIs) for Availability
Availability is often calculated as:
$$\text{Availability} = \frac{\text{Successful Requests}}{\text{Total Valid Requests}}$$
*   **Success**: HTTP 2xx or 3xx.
*   **Failure**: HTTP 5xx (Server error) or timeouts. Note: 4xx errors are usually excluded as they indicate user error (e.g., wrong password).

### 4. Comparison Table

| Feature | Uptime Checks | Synthetic Transactions |
| :--- | :--- | :--- |
| **Complexity** | Low (Single URL) | High (Multi-step script) |
| **Cost** | Very Cheap | Expensive (Compute intensive) |
| **Detection** | "Is the server up?" | "Is the business logic working?" |
| **Frequency** | Every 30-60 seconds | Every 5-15 minutes |

## Go-specific Applications
In Go, availability monitoring can be complemented by a "Heartbeat" service that periodically verifies downstream dependencies.

### Heartbeat Pattern
**Evidence** ([source](https://github.com/moby/moby/blob/master/api/server/router/system/system.go)):
```go
// Example of a diagnostic check that runs internally
func (s *SystemRouter) getPing(ctx context.Context, w http.ResponseWriter, r *http.Request, vars map[string]string) error {
    if err := s.backend.SystemPing(ctx); err != nil {
        return err
    }
    w.Header().Set("Cache-Control", "no-cache, no-store, must-revalidate")
    w.WriteHeader(http.StatusOK)
    w.Write([]byte("OK"))
    return nil
}
```

## Interview Preparation Questions

**Q1: What is "Partial Availability" and how do you monitor it?**
**A1:** Partial availability occurs when a service is up in some regions but down in others, or when some features (e.g., Search) are broken while others (e.g., Login) work. Global uptime checks from multiple PoPs (Points of Presence) and feature-specific synthetic scripts are required to detect this.

**Q2: Why is Black-box monitoring better for alerting than White-box?**
**A2:** Black-box monitoring is symptom-oriented. It only pings the SRE if the user is actually seeing an error. White-box monitoring (e.g., high CPU) might be a "false alarm" if the system is still serving requests perfectly fine under load.

**Q3: How do you exclude maintenance windows from availability metrics?**
**A3:** Most monitoring tools (Datadog, New Relic) allow for "Maintenance Windows" or "Muting" during which alerts are suppressed and downtime is not counted against the monthly SLO.
