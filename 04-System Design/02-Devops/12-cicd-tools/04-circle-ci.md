---
---

# CircleCI

CircleCI is a cloud-based CI/CD platform known for its speed and developer-centric features. It emphasizes performance optimization through caching, parallelism, and reusable configuration packages called **Orbs**.

## Summary

CircleCI uses a `config.yml` (version 2.1) to define workflows. It operates primarily as a SaaS (cloud) offering but has a self-hosted option. Its architecture is built around **Docker** containers as first-class citizens, making it very fast for containerized workflows.

## Detailed Explanation

### 1. Key Concepts
*   **Orbs**: Shareable packages of configuration (e.g., `circleci/aws-s3`). They reduce boilerplate significantly.
*   **Workflows**: Define the orchestration of jobs (parallel, sequential, fan-in/fan-out).
*   **Contexts**: Mechanism to share environment variables securely across projects.
*   **Test Splitting**: Automatically splits test suites across multiple parallel containers based on timing data to reduce total build time.

---

## Go Implementation Example

Using the `jszwedko/go-circleci` library to interact with the CircleCI API v2. This is useful for triggering pipelines or inspecting build insights.

```go
package main

import (
	"fmt"
	"log"

	"github.com/jszwedko/go-circleci"
)

func main() {
	token := "YOUR_CIRCLECI_TOKEN"
	client := &circleci.Client{Token: token}

	// 1. Trigger a Pipeline
	// VCS type (github/bitbucket), Org, Repo
	account := "my-org"
	repo := "my-go-app"
	
	build, err := client.TriggerBuild("github", account, repo, "main", nil)
	if err != nil {
		log.Fatalf("Error triggering build: %v", err)
	}

	fmt.Printf("Build triggered! URL: %s\n", build.BuildURL)

	// 2. Get recent builds
	builds, err := client.ListRecentBuildsForProject("github", account, repo, "", "", 5, 0)
	if err != nil {
		log.Fatal(err)
	}

	for _, b := range builds {
		fmt.Printf("Build #%d - Status: %s\n", b.BuildNum, b.Status)
	}
}
```

## Interview Questions

**Q: What are "Orbs" in CircleCI?**
**A:** Orbs are reusable snippets of configuration code (YAML) that can be shared across projects. They abstract away complex commands. For example, instead of writing 20 lines of AWS CLI commands to upload to S3, you can use the `aws-s3` Orb and write `aws-s3/sync`.

**Q: How does CircleCI achieve parallelism?**
**A:** CircleCI allows you to set a `parallelism` level for a job. It spins up N identical containers. You can then use the CircleCI CLI to split your test files (`go test $(circleci tests split files_list)`) so that each container runs a fraction of the tests, drastically reducing the time to feedback.

**Q: What is the difference between "Caching" and "Workspaces" in CircleCI?**
**A:**
*   **Caching**: Persists data between *different runs* of the same workflow (e.g., `go-build-cache`). Used for dependency management.
*   **Workspaces**: Persists data between *jobs* in the *same workflow* run. Used to pass built binaries from a "Build" job to a "Deploy" job.
