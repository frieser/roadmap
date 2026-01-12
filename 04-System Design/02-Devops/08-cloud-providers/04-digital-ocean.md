---
---

# DigitalOcean

DigitalOcean (DO) is a cloud provider focused on simplicity and developer experience. It is popular among startups, SMBs, and individual developers who find AWS/Azure/GCP too complex or expensive for their needs.

## Summary

DO simplifies cloud infrastructure into easy-to-understand primitives. Its pricing is transparent (flat monthly rates), and its UI is intuitive. Core offerings include **Droplets** (VMs), **Managed Kubernetes**, and **App Platform** (PaaS). It is an excellent choice for straightforward deployments where "hyperscale" features are not required.

## Detailed Explanation

### 1. Key Concepts
*   **Droplets**: Virtual Machines.
*   **Spaces**: S3-compatible object storage.
*   **App Platform**: A PaaS solution (similar to Heroku) that builds and deploys code directly from GitHub.
*   **Floating IPs**: Static IPs that can be instantly remapped between Droplets.

### 2. DevOps on DigitalOcean
While simpler, DO supports modern DevOps practices:
*   **Terraform Provider**: Fully supported.
*   **Doctl**: A powerful CLI tool.
*   **Cloud-Init**: Support for bootstrapping Droplets.

---

## Go Implementation Example

DigitalOcean provides `godo`, a comprehensive Go client library.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/digitalocean/godo"
)

func main() {
	// Token from environment
	token := "YOUR_DIGITALOCEAN_TOKEN"

	// 1. Create Client
	client := godo.NewFromToken(token)
	ctx := context.TODO()

	// 2. Define Droplet Request
	createRequest := &godo.DropletCreateRequest{
		Name:   "go-devops-droplet",
		Region: "nyc3",
		Size:   "s-1vcpu-1gb",
		Image:  godo.DropletCreateImage{Slug: "ubuntu-22-04-x64"},
		Tags:   []string{"devops", "test"},
	}

	// 3. Create Droplet
	droplet, _, err := client.Droplets.Create(ctx, createRequest)
	if err != nil {
		log.Fatalf("Something bad happened: %s\n", err)
	}

	fmt.Printf("Droplet created!\nID: %d\nName: %s\nRegion: %s\n", 
		droplet.ID, droplet.Name, droplet.Region.Slug)
}
```

## Interview Questions

**Q: When would you use DigitalOcean over AWS?**
**A:** Use DigitalOcean when you need predictable pricing (flat monthly fee vs. complex hourly/usage metrics), simplicity (launching a VM takes seconds with minimal config), or when the project is a standard web application that doesn't require specialized proprietary services (like DynamoDB or Kinesis).

**Q: What is a "Floating IP" and how does it help with High Availability?**
**A:** A Floating IP is a static public IP address that can be remapped from one Droplet to another instantly via API. This allows for a basic High Availability (HA) setup: if your primary web server fails, a monitoring script (or a human) can reassign the Floating IP to a standby server, minimizing downtime without waiting for DNS propagation.

**Q: Is DigitalOcean's Object Storage (Spaces) compatible with S3 tools?**
**A:** Yes, Spaces is S3-compatible. This means you can use the standard AWS SDKs (including `aws-sdk-go-v2`) or CLI tools (like `s3cmd`) to interact with Spaces simply by changing the Endpoint URL to DigitalOcean's region URL (e.g., `nyc3.digitaloceanspaces.com`).
