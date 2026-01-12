---
---

# Google Cloud Platform (GCP)

GCP is Google's suite of cloud computing services. It is renowned for its data analytics capabilities (BigQuery), its premier Kubernetes support (GKE), and its high-performance global network.

## Summary

GCP is often considered the most "developer-friendly" cloud due to its cleaner console and faster startup times compared to AWS. Its core strengths lie in **Containers** (Google invented Kubernetes), **Data/AI** (BigQuery, Vertex AI), and **Networking** (Global VPCs). The Go SDK is first-class, as Go was born at Google.

## Detailed Explanation

### 1. Core Services
*   **GCE (Compute Engine)**: Virtual Machines. Known for fast boot times and custom machine types (custom CPU/RAM ratios).
*   **GCS (Cloud Storage)**: Object storage (S3 equivalent). Consistent API.
*   **GKE (Google Kubernetes Engine)**: The gold standard for managed Kubernetes.
*   **BigQuery**: Serverless, highly scalable data warehouse.

### 2. Networking Difference
In AWS, a VPC is regional. In GCP, a **VPC is Global**. Subnets are regional. This simplifies multi-region architecture significantly, as you don't need complex peering setups to talk between regions within the same VPC.

---

## Go Implementation Example

Google provides the `cloud.google.com/go` library. It feels very native to Go developers.

```go
package main

import (
	"context"
	"fmt"
	"io"
	"log"
	"os"

	"cloud.google.com/go/storage"
)

func main() {
	ctx := context.Background()
	bucketName := "my-devops-bucket"
	objectName := "test-upload.txt"

	// 1. Create Client (Authenticates via GOOGLE_APPLICATION_CREDENTIALS)
	client, err := storage.NewClient(ctx)
	if err != nil {
		log.Fatalf("Failed to create client: %v", err)
	}
	defer client.Close()

	// 2. Open local file
	f, err := os.Open("local-file.txt")
	if err != nil {
		// Create a dummy file for example
		f, _ = os.CreateTemp("", "example")
		f.WriteString("Hello GCP!")
		f.Seek(0, 0)
	}
	defer f.Close()

	// 3. Upload Object
	wc := client.Bucket(bucketName).Object(objectName).NewWriter(ctx)
	if _, err = io.Copy(wc, f); err != nil {
		log.Fatalf("io.Copy: %v", err)
	}
	if err := wc.Close(); err != nil {
		log.Fatalf("Writer.Close: %v", err)
	}

	fmt.Printf("File uploaded to gs://%s/%s\n", bucketName, objectName)
}
```

## Interview Questions

**Q: Why is GKE often preferred over EKS or AKS?**
**A:** GKE is widely considered the most mature managed Kubernetes offering because Kubernetes was designed by Google. It often gets new K8s versions and features first. It provides an automated "Autopilot" mode that fully abstracts node management, allowing DevOps teams to focus purely on Pods.

**Q: Explain the concept of a "Global VPC" in GCP.**
**A:** In most clouds (AWS/Azure), a Virtual Network is bound to a specific Region. If you want resources in US-East to talk to EU-West, you need VPC Peering or a VPN. In GCP, a VPC spans the entire globe. You can have a subnet in `us-central1` and another in `europe-west1` inside the *same* VPC, and they can communicate via internal IP addresses automatically over Google's backbone.

**Q: What is "Preemptible VMs" (or Spot VMs)?**
**A:** These are short-lived compute instances sold at a massive discount (up to 80%). However, Google can reclaim them at any time if it needs the capacity. They are ideal for fault-tolerant batch processing or stateless workloads (like GKE nodes) that can handle sudden interruptions.
