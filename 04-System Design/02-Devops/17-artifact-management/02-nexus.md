---
---

# Sonatype Nexus Repository

Sonatype Nexus is one of the most popular repository managers, particularly strong in the Java ecosystem (Maven) but supporting all major formats including Docker, npm, PyPI, and Go.

## Summary

Nexus Repository Manager (NXRM) allows you to proxy, collect, and manage your dependencies so that you are not constantly dealing with external repos. It acts as a dedicated server for managing binary artifacts. It offers both an OSS (Open Source) version and a Pro version with advanced features like High Availability.

## Detailed Explanation

### 1. Key Concepts
*   **Blob Stores**: The storage mechanism used by Nexus. Can be a local filesystem or AWS S3.
*   **Format-Specific Features**: Nexus understands the metadata of each format. For Docker, it supports the Docker Registry V2 API. For Maven, it handles snapshots and releases.
*   **Staging**: A Pro feature that allows you to move artifacts through a lifecycle (e.g., Build -> Test -> Release) with validation gates.

### 2. Architecture
Nexus is a Java application. It uses an internal database (OrientDB in older versions, H2/PostgreSQL in newer 3.x) to manage metadata.

---

## Go Implementation Example

While Nexus has a robust REST API, Go developers often interact with it simply as a **GOPROXY**. However, for automation (e.g., creating repositories programmatically), you can use the REST API.

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"net/http"
)

func main() {
	// Create a new Raw (Generic) Repository via API
	nexusURL := "http://localhost:8081/service/rest/v1/repositories/raw/hosted"
	
	payload := map[string]interface{}{
		"name": "my-go-binaries",
		"online": true,
		"storage": map[string]interface{}{
			"blobStoreName": "default",
			"strictContentTypeValidation": true,
			"writePolicy": "ALLOW",
		},
	}
	
	jsonPayload, _ := json.Marshal(payload)
	req, _ := http.NewRequest("POST", nexusURL, bytes.NewBuffer(jsonPayload))
	req.SetBasicAuth("admin", "admin123")
	req.Header.Set("Content-Type", "application/json")

	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()

	if resp.StatusCode == 201 {
		fmt.Println("Repository created successfully!")
	} else {
		fmt.Printf("Failed: %s\n", resp.Status)
	}
}
```

## Interview Questions

**Q: What is a "Snapshot" version in Nexus (Maven context)?**
**A:** A Snapshot version (e.g., `1.0.0-SNAPSHOT`) is a mutable artifact. It represents a version currently under development. Unlike Release versions (which are immutable), you can upload a new Snapshot with the same version string, and Nexus will handle the timestamping so clients get the latest build.

**Q: How does Nexus handle Docker images differently from Maven artifacts?**
**A:** Docker images are composed of Layers (Blobs) and Manifests. Nexus stores layers in the Blob Store. Since layers are often shared between images (deduplication), Nexus must carefully manage garbage collection to ensure it doesn't delete a layer used by another image. It also supports the Docker V2 API to act as a standard registry.

**Q: What is the "Group Repository" in Nexus?**
**A:** It is the Nexus terminology for a **Virtual Repository**. It groups a collection of Hosted (Local) and Proxy (Remote) repositories into a single URL for easy consumption by build tools.
