---
---

## Summary
While managing your own Kubernetes cluster offers maximum control, it comes with high operational overhead ("Day 2 operations"). Platform-as-a-Service (PaaS) options abstract away the infrastructure management, allowing developers to focus solely on code and containers.

## Detailed Explanation

### Options Landscape
1.  **Google Cloud Run**: Serverless containers. Scale to zero. Pay per request. Great for stateless HTTP services.
2.  **AWS Fargate**: Serverless engine for ECS/EKS. You define CPU/RAM, AWS manages the underlying EC2s.
3.  **Heroku / Dokku**: The classic "git push to deploy". Highly opinionated, easy to start, expensive at scale.
4.  **Azure Container Apps**: Similar to Cloud Run, based on KEDA and Envoy.

### Trade-offs
*   **PaaS**: High ease of use, auto-scaling built-in, no OS patching. **Cons**: Vendor lock-in, cold starts (Serverless), cost limits on high traffic.
*   **K8s**: Infinite flexibility, cheaper at massive scale. **Cons**: Requires dedicated DevOps team to manage upgrades and security.

## Go-Specific Context/Examples

Go's fast startup time makes it **perfect** for Serverless PaaS like Cloud Run (Cold starts < 100ms).

### Example: Cloud Run Ready Go Server
Simply listen on the `PORT` env var.

```go
package main

import (
	"fmt"
	"net/http"
	"os"
)

func main() {
	port := os.Getenv("PORT")
	if port == "" {
		port = "8080"
	}
	
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "Hello from Cloud Run!")
	})
	
	fmt.Printf("Listening on %s...\n", port)
	http.ListenAndServe(":"+port, nil)
}
```
Dockerfile: `FROM golang:alpine ... CMD ["/app"]`.
Deploy: `gcloud run deploy --source .`

## Interview Questions

**Q: What is a "Cold Start"?**
**A:** When a serverless platform (Cloud Run/Lambda) scales from 0 to 1 instance, it must provision a container and start the process. The latency added by this startup is the cold start penalty.

**Q: Why migrate from Heroku to Kubernetes?**
**A:** Cost and Control. Heroku gets very expensive for large deployments. K8s allows fine-grained control over networking, persistent volumes, and custom sidecars that Heroku might restrict.

**Q: Can you run stateful apps (DB) on Cloud Run?**
**A:** Technically yes (connecting to Cloud SQL), but Cloud Run instances are ephemeral and stateless. You cannot store data on the local filesystem and expect it to persist.
