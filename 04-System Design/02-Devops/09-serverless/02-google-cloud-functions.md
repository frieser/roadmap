---
---

# Google Cloud Functions

Google Cloud Functions (GCF) is Google's event-driven serverless compute platform. It is highly scalable and integrates deeply with Google's data analytics and Firebase ecosystems.

## Summary

GCF comes in two generations. **2nd Gen** is the modern standard, built on top of **Cloud Run** and **Eventarc**, offering longer execution times, larger instance sizes, and traffic splitting. Go is natively supported.

## Detailed Explanation

### 1. Gen 1 vs Gen 2
*   **Gen 1**: Proprietary architecture. Good for simple triggers.
*   **Gen 2**: Built on Cloud Run (Knative).
    *   **Concurrency**: Can handle multiple concurrent requests per instance (unlike AWS Lambda which is 1 request per instance).
    *   **Eventarc**: A unified eventing system that lets you trigger functions from over 90+ Google Cloud sources.

### 2. The Google Functions Framework
Google provides a standardized library (`github.com/GoogleCloudPlatform/functions-framework-go`) that allows you to write functions that are portable. You can run the exact same function locally, in Cloud Functions, or in Cloud Run/Knative.

---

## Go Implementation Example

Using the Functions Framework for an HTTP function.

```go
package helloworld

import (
	"fmt"
	"net/http"

	"github.com/GoogleCloudPlatform/functions-framework-go/functions"
)

func init() {
	// Register the function to handle HTTP requests
	functions.HTTP("HelloHTTP", helloHTTP)
}

// helloHTTP is an HTTP Cloud Function
func helloHTTP(w http.ResponseWriter, r *http.Request) {
	name := r.URL.Query().Get("name")
	if name == "" {
		name = "World"
	}
	fmt.Fprintf(w, "Hello, %s!", name)
}
```

### Local Development
You can run this locally without deploying:
```bash
go run cmd/main.go # (Boilerplate main provided by framework)
curl http://localhost:8080?name=Go
```

## Interview Questions

**Q: What is the main architectural difference between GCF Gen 1 and Gen 2?**
**A:** Gen 2 is built on top of **Cloud Run** (and by extension, Knative). This means a Gen 2 function is actually a containerized service. This architecture unlocks features like processing **concurrent requests** on a single instance (reducing cold starts for high-traffic APIs) and traffic splitting (Canary deployments), which were difficult or impossible in Gen 1.

**Q: How does Eventarc change the trigger model in GCF?**
**A:** Eventarc standardizes event delivery using the **CloudEvents** specification. Instead of bespoke proprietary events for every service, Eventarc provides a uniform way to filter and route events (e.g., "Image uploaded to Bucket A" or "BigQuery Job Finished") to your function, often handling the delivery via Pub/Sub under the hood.

**Q: Can you access a VPC resource (like a Redis instance) from a Cloud Function?**
**A:** Yes, via the **Serverless VPC Access Connector**. You deploy a connector in your VPC, and configure your Cloud Function to route egress traffic through it. This allows the function to reach private IPs (like Cloud Memorystore or a private SQL instance) without exposing them to the public internet.
