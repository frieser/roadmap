#API
---
---

# IDS, IPS, and System Monitoring

## Summary
Intrusion Detection Systems (IDS) and Intrusion Prevention Systems (IPS) are critical for identifying and mitigating threats in API environments. IDS acts as a passive observer that alerts on suspicious activity, while IPS actively blocks malicious traffic. Complemented by System Monitoring (observing CPU, Memory, and Network metrics), these tools provide a holistic view of API health and security, ensuring availability and protecting against attacks like DDoS, credential stuffing, and injection.

---

## Detailed Explanation

### 1. Intrusion Detection vs. Prevention (IDS/IPS)
The primary difference between IDS and IPS is the **action** taken upon detection.

*   **Intrusion Detection System (IDS)**: Monitors network traffic and system logs for suspicious activity. It generates alerts but does not stop the activity.
    *   **HIDS (Host-based)**: Monitors a single host (e.g., OSSEC).
    *   **NIDS (Network-based)**: Monitors network traffic (e.g., Snort, Suricata).
*   **Intrusion Prevention System (IPS)**: Sits in-line with traffic and can actively block or drop packets that match attack signatures or deviate from normal behavior.

**Detection Methods:**
1.  **Signature-based**: Matches patterns against a database of known threats. Effective for known vulnerabilities but fails against zero-day attacks.
2.  **Anomaly-based**: Uses a baseline of "normal" behavior and alerts on deviations. Better for unknown threats but prone to false positives.

### 2. System Monitoring Metrics for Security
Monitoring system health is a secondary line of defense for API security. Anomalies in resource usage often indicate an ongoing attack.

*   **CPU Usage**: Spikes may indicate crypto-mining malware or complex injection attacks.
*   **Memory Usage**: Sudden increases can signal buffer overflow attempts or memory leaks from DoS attacks.
*   **Disk I/O**: High activity might indicate unauthorized data exfiltration or log tampering.
*   **Network Throughput**: Massive surges usually point to DDoS or large-scale data scraping.

**The Golden Signals of Monitoring:**
*   **Latency**: Time it takes to service a request.
*   **Traffic**: Demand placed on the system.
*   **Errors**: Rate of requests that fail.
*   **Saturation**: How "full" your service is.

### 3. Key Tools
*   **Snort/Suricata**: Industry-standard NIDS/IPS tools for deep packet inspection.
*   **OSSEC**: A powerful HIDS for log analysis, integrity checking, and rootkit detection.
*   **Prometheus**: A time-series database used for gathering metrics via a pull model.
*   **Grafana**: A visualization platform to create dashboards for Prometheus data.

---

## Go (Golang) Application

In Go, security monitoring is often implemented through health check endpoints and Prometheus instrumentation.

### Basic Health Check Implementation
A simple health check allows load balancers and monitoring tools to verify if the API is "alive".

**Evidence** ([source](https://github.com/kubernetes/ingress-gce/blob/master/cmd/echo/app/handlers.go#L93-L98)):
```go
func healthCheck(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusOK)
	_, err := w.Write([]byte("health: OK"))
	if err != nil {
		klog.Errorf("Error writing bytes: %v", err)
		return
	}
}
```

### Prometheus Instrumentation
Using `prometheus/client_golang` to track request counts, which can help detect brute-force attacks.

**Evidence** ([source](https://github.com/milvus-io/milvus/blob/master/pkg/metrics/proxy_metrics.go#L26-L32)):
```go
import (
	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promhttp"
	"net/http"
)

var (
	// Track received search and query requests
	ProxyReceivedRequests = prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Namespace: "api",
			Name:      "received_requests_total",
			Help:      "Total number of received API requests",
		}, []string{"method", "endpoint", "status"})
)

func init() {
	prometheus.MustRegister(ProxyReceivedRequests)
}

// Middleware to record metrics
func metricsMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Logic to track request status...
		ProxyReceivedRequests.WithLabelValues(r.Method, r.URL.Path, "200").Inc()
		next.ServeHTTP(w, r)
	})
}
```

---

## Interview Questions

**Q1: What is the main difference between IDS and IPS?**
**A:** IDS is passive (detects and alerts), while IPS is active (detects and blocks). IDS is like a security camera; IPS is like a security guard.

**Q2: How can high CPU usage indicate a security breach?**
**A:** Spikes in CPU can indicate computationally expensive attacks like password cracking (brute force), XML External Entity (XXE) processing, or the presence of malware like cryptojackers.

**Q3: Explain "False Positives" vs "False Negatives" in the context of IDS.**
**A:** A **False Positive** is when the system incorrectly flags legitimate traffic as an attack. A **False Negative** is when an actual attack passes through the system undetected.

**Q4: Why is anomaly-based detection useful for zero-day attacks?**
**A:** Zero-day attacks have no known signatures. Anomaly-based detection focuses on behavior; if a zero-day attack causes a system to behave unusually (e.g., massive outbound traffic), it can be detected even without a signature.

**Q5: What are the "Four Golden Signals" in monitoring?**
**A:** Latency, Traffic, Errors, and Saturation. These provide a comprehensive overview of system performance and can reveal underlying security or stability issues.
