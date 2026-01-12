---
---

# JFrog Artifactory & Sonatype Nexus

In the DevOps toolchain, an **Artifact Repository Manager** acts as the single source of truth for all binary artifacts (JARs, NPM packages, Docker images, Go modules). It bridges the gap between Continuous Integration (Build) and Continuous Deployment (Release).

## Summary

*   **JFrog Artifactory**: The "Universal" repository manager. Supports almost every package format. Known for its powerful metadata query language (AQL) and "Virtual Repositories" (aggregating local and remote repos).
*   **Sonatype Nexus**: A strong alternative, often used in Java-heavy shops. The OSS version is widely deployed. Focuses on component lifecycle management and supply chain security (Firewall).

## Detailed Explanation

### 1. Repository Types
*   **Local Repository**: Stores artifacts built internally by your CI/CD pipelines.
*   **Remote Repository**: A caching proxy for external sources (e.g., Maven Central, npmjs.org, Docker Hub). Improves build speed and reliability.
*   **Virtual Repository**: A single URL that aggregates multiple local and remote repositories. Developers configure their tools to point to this one URL.

### 2. DevOps Use Cases
*   **Go Modules Proxy**: Configuring `GOPROXY` to point to Artifactory ensures that your build is reproducible and safe from upstream deletions ("Left-pad incident").
*   **Docker Registry**: Hosting private container images with fine-grained access control.

---

## Go Implementation Example

Using the Artifactory REST API to upload a binary. This is a common step in a custom CI pipeline written in Go.

```go
package main

import (
	"fmt"
	"os"

	"github.com/jfrog/jfrog-client-go/artifactory"
	"github.com/jfrog/jfrog-client-go/artifactory/auth"
	"github.com/jfrog/jfrog-client-go/artifactory/services"
	"github.com/jfrog/jfrog-client-go/config"
)

func main() {
	// 1. Configure Authentication
	rtDetails := auth.NewArtifactoryDetails()
	rtDetails.SetUrl("https://my-artifactory.com/artifactory/")
	rtDetails.SetUser("admin")
	rtDetails.SetPassword("password")

	serviceConfig, _ := config.NewConfigBuilder().
		SetServiceDetails(rtDetails).
		Build()

	// 2. Create Client
	rtManager, _ := artifactory.New(serviceConfig)

	// 3. Define Upload Parameters
	params := services.UploadParams{
		Repo:           "my-go-local",
		SourcePattern:  "./my-binary",
		TargetProps:    "build.name=my-app;build.number=101", // Metadata
	}

	// 4. Perform Upload
	// This replaces "curl -u user:pass -T my-binary ..."
	totalUploaded, totalFailed, err := rtManager.UploadFiles(params)
	if err != nil {
		fmt.Printf("Upload error: %v\n", err)
		os.Exit(1)
	}

	fmt.Printf("Uploaded %d files. Failed: %d\n", totalUploaded, totalFailed)
}
```

## Interview Questions

**Q: What is the difference between a Proxy Repository and a Hosted Repository?**
**A:**
*   **Hosted (Local)**: You upload your own artifacts here (e.g., your company's private libraries).
*   **Proxy (Remote)**: You do not upload here. It proxies a public repository (like npmjs.org). When you request a package, it downloads it from the internet and caches it locally. Subsequent requests are served from the cache.

**Q: Why is "GOPROXY" important for Go development in an enterprise?**
**A:** By default, `go get` fetches code directly from VCS (GitHub, GitLab). This is slow and risky (authors can delete repos). Setting `GOPROXY` to an internal Artifact Manager ensures:
1.  **Immutability**: Builds are reproducible even if the source repo vanishes.
2.  **Speed**: Faster downloads from the local network.
3.  **Security**: You can block vulnerable packages at the proxy level.

**Q: What is a "Virtual Repository"?**
**A:** A Virtual Repository aggregates multiple Local and Remote repositories under a single URL. This simplifies client configuration. A developer sets their NPM registry to `http://artifactory/api/npm/virtual-repo`, and they can seamlessly install both public packages (from the proxied remote) and private internal packages (from the local repo).
