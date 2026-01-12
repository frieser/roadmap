---
---

# Heroku

Heroku is a container-based cloud Platform as a Service (PaaS). It was one of the first platforms to popularize the "git push to deploy" workflow. It abstracts away almost all infrastructure management, allowing developers to focus solely on application code.

## Summary

Heroku runs applications in isolated Linux containers called **Dynos**. It pioneered the **12-Factor App** methodology. While more expensive per unit of compute than raw AWS EC2, the savings in DevOps man-hours make it valuable for rapid prototyping and startups. It supports Go natively via Buildpacks.

## Detailed Explanation

### 1. Key Concepts
*   **Dynos**: The unit of compute. You scale vertically (bigger dynos) or horizontally (more dynos) with a slider.
*   **Buildpacks**: Scripts that detect your language (e.g., seeing a `go.mod` file), install dependencies, and compile the app.
*   **Procfile**: A text file in the root of your repo that tells Heroku how to start your app.
*   **Add-ons**: A marketplace of managed services (Postgres, Redis, Logging) that can be provisioned and linked to your app with one click.

### 2. The Deployment Workflow
1.  Developer commits code.
2.  `git push heroku main`
3.  Heroku detects Go, builds the binary, creates a "slug" (compressed filesystem), and deploys it to Dynos.

---

## Go Implementation Example

Heroku deployments don't typically use an SDK for the *app logic itself*; the app just needs to be a standard web server listening on the `PORT` environment variable. However, you can use the Heroku Platform API to manage the infrastructure programmatically.

### 1. The Application (main.go)
```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"os"
)

func main() {
	// Heroku injects the PORT environment variable
	port := os.Getenv("PORT")
	if port == "" {
		port = "8080" // Fallback for local dev
	}

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Hello from Heroku!")
	})

	log.Printf("Listening on port %s", port)
	log.Fatal(http.ListenAndServe(":"+port, nil))
}
```

### 2. The Procfile
```yaml
web: ./my-app-binary
```

### 3. Managing Heroku via Go (Platform API)
Using `github.com/heroku/heroku-go/v5`.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/heroku/heroku-go/v5"
)

func main() {
	// Authenticate
	h := heroku.NewService(heroku.DefaultClient)
	
	// List Apps
	apps, err := h.AppList(context.Background(), &heroku.ListRange{Field: "name"})
	if err != nil {
		log.Fatal(err)
	}

	for _, app := range apps {
		fmt.Printf("App: %s (Region: %s)\n", app.Name, app.Region.Name)
	}
	
	// Scale a Dyno (Conceptual)
	// h.DynoUpdate(ctx, "app-name", "web.1", ...)
}
```

## Interview Questions

**Q: What is a "Procfile"?**
**A:** A `Procfile` is a mechanism for declaring what commands are run by your application's dynos on the Heroku platform. For a web app, it typically looks like `web: ./bin/server`. It allows you to define multiple process types (e.g., `worker: ./bin/worker`) in a single repo.

**Q: Why do Heroku Dynos restart automatically once a day?**
**A:** This is a feature called "cycling." It ensures that the application stays healthy and doesn't accumulate memory leaks or file system cruft (since the filesystem is ephemeral). It also allows Heroku to patch the underlying OS and move dynos to different physical hardware without downtime (due to load balancing).

**Q: What are "Ephemereal Filesystems" in the context of Heroku?**
**A:** Each dyno gets its own ephemeral filesystem. You can write files to disk, but if the dyno restarts (which happens daily or on deploy), **those files are lost**. This enforces the 12-Factor principle of treating backing services (like S3) as attached resources. You should never store user uploads on the local disk in Heroku; always upload them to S3.
