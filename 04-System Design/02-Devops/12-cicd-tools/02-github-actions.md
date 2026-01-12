---
---

# GitHub Actions

GitHub Actions is a cloud-native CI/CD platform integrated directly into GitHub. It has rapidly become a favorite due to its ease of use, massive marketplace of reusable actions, and tight integration with the repository lifecycle.

## Summary

GitHub Actions relies on **Workflows** defined in YAML files located in `.github/workflows`. Workflows are triggered by events (Push, PR, Release). Jobs run on **Runners** (Ubuntu, Windows, macOS) provided by GitHub or self-hosted.

## Detailed Explanation

### 1. Key Concepts
*   **Workflow**: The automated process (YAML file).
*   **Event**: What triggers the workflow (e.g., `on: push`).
*   **Job**: A set of steps executing on the same runner. Jobs run in parallel by default.
*   **Step**: A shell command (`run: go test`) or an Action (`uses: actions/checkout@v4`).
*   **Action**: Reusable code unit. Can be a JavaScript app or a Docker container.

### 2. DevOps Strengths
*   **Matrix Builds**: Easily run tests across Go 1.20, 1.21, and 1.22 in parallel.
*   **Security**: OIDC integration allows authenticated access to AWS/Azure/GCP without storing long-lived keys.

---

## Go Implementation Example

You can write **Custom GitHub Actions** in Go. Since runners don't always have the Go toolchain you want, you typically compile your Go action into a static binary or a Docker container.

### Custom Action Logic (`main.go`)
This action reads an input and sets an output variable.

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// 1. Inputs are passed as environment variables: INPUT_<NAME>
	name := os.Getenv("INPUT_WHO_TO_GREET")
	if name == "" {
		name = "World"
	}

	fmt.Printf("Hello, %s!\n", name)

	// 2. Set Output using the GITHUB_OUTPUT environment file
	// Old way (::set-output) is deprecated.
	outputFile := os.Getenv("GITHUB_OUTPUT")
	f, err := os.OpenFile(outputFile, os.O_APPEND|os.O_WRONLY, 0644)
	if err == nil {
		defer f.Close()
		// Format: key=value
		fmt.Fprintf(f, "time=%s\n", "some-timestamp")
	}
}
```

## Interview Questions

**Q: How does GitHub Actions billing work for public vs private repositories?**
**A:** For **public** repositories, GitHub Actions is completely free (standard runners). For **private** repositories, you get a monthly allowance of minutes (e.g., 2000 mins for Pro) and pay-as-you-go thereafter. Self-hosted runners are free to use.

**Q: What is the `GITHUB_TOKEN`?**
**A:** It is a temporary, automatically generated authentication token created for each workflow run. It allows the workflow to interact with the repository (e.g., creating releases, adding comments to PRs) without needing a personal access token. Its permissions can be configured in the YAML file.

**Q: How do you share data between jobs in GitHub Actions?**
**A:** Since jobs run on different runners, they do not share a filesystem. To share data (e.g., a built binary), you must use **Artifacts**. Job A uses `actions/upload-artifact` to save the file, and Job B uses `actions/download-artifact` to retrieve it.
