---
---

# Jenkins

Jenkins is the granddaddy of CI/CD. It is an open-source, self-hosted automation server written in Java. Despite its age, its massive plugin ecosystem (1800+) makes it incredibly flexible for complex enterprise pipelines.

## Summary

Jenkins allows you to define pipelines as code using **Jenkinsfiles** (Groovy DSL). It uses a **Controller-Agent** architecture where the Controller manages scheduling and the Agents (Linux, Windows, Docker) execute the workloads.

## Detailed Explanation

### 1. Architecture
*   **Controller (Master)**: Stores configuration, loads plugins, and schedules builds.
*   **Agents (Nodes)**: Worker machines that run the jobs. Labels allow targeting specific OSs (e.g., `label 'windows'`).
*   **Plugins**: Extend functionality. Everything from Git integration to Kubernetes scaling is a plugin.

### 2. Jenkinsfile (Pipeline as Code)
*   **Declarative Pipeline**: Structured, simpler syntax (`pipeline { agent any ... }`). Preferred for most use cases.
*   **Scripted Pipeline**: Full Groovy power. Used for highly complex logic.

---

## Go Implementation Example

Using the `bndr/gojenkins` library to interact with the Jenkins API. This is useful for building custom dashboards or triggering builds from other tools.

```go
package main

import (
	"context"
	"fmt"
	"log"
	"time"

	"github.com/bndr/gojenkins"
)

func main() {
	ctx := context.Background()
	// Connect to Jenkins
	jenkins := gojenkins.CreateJenkins(nil, "http://localhost:8080", "admin", "api-token")
	_, err := jenkins.Init(ctx)
	if err != nil {
		log.Fatalf("Connection failed: %v", err)
	}

	// Trigger a Job
	jobName := "my-go-pipeline"
	queueid, err := jenkins.BuildJob(ctx, jobName, map[string]string{"BRANCH": "main"})
	if err != nil {
		log.Fatalf("Build failed: %v", err)
	}

	fmt.Printf("Job triggered! Queue ID: %d\n", queueid)

	// Poll for build completion (Simplified)
	time.Sleep(10 * time.Second)
	job, _ := jenkins.GetJob(ctx, jobName)
	lastBuild, _ := job.GetLastBuild(ctx)
	fmt.Printf("Build Number: %d, Result: %s\n", lastBuild.GetBuildNumber(), lastBuild.GetResult())
}
```

## Interview Questions

**Q: What is the difference between Declarative and Scripted Pipelines?**
**A:**
*   **Declarative**: Provides a stricter, pre-defined structure. It includes syntax for error handling (`post`), environment variables, and agent definition. Easier to read and maintain.
*   **Scripted**: Essentially a Groovy script. Offers extreme flexibility (loops, complex logic) but is harder to maintain and visualize in the UI.

**Q: Why is it recommended to not run builds on the Jenkins Controller?**
**A:** Running builds on the Controller ("Master") poses security risks (jobs have access to master credentials) and performance risks (heavy builds can crash the UI/scheduler). Best practice is to set "Number of executors" to 0 on the Controller and delegate all work to Agents.

**Q: How do you handle secrets in Jenkins?**
**A:** Use the **Credentials Binding Plugin**. Secrets are stored securely in Jenkins' internal store. In the Jenkinsfile, you use the `credentials()` helper to inject them as environment variables (e.g., `APP_KEY`) into the build scope only when needed. They are masked in the logs.
