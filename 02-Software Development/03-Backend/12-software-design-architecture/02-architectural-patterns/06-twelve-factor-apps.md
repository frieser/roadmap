---
---

## Summary
The 12-Factor App is a methodology for building software-as-a-service applications that are portable, resilient, and scalable. It provides a set of best practices for modern cloud-native development. Go's design and runtime characteristics make it a natural fit for building 12-factor apps.

## Detailed Explanation

### The 12 Factors
1.  **Codebase**: One codebase tracked in revision control, many deploys.
2.  **Dependencies**: Explicitly declare and isolate dependencies.
3.  **Config**: Store config in the environment.
4.  **Backing services**: Treat backing services as attached resources.
5.  **Build, release, run**: Strictly separate build and run stages.
6.  **Processes**: Execute the app as one or more stateless processes.
7.  **Port binding**: Export services via port binding.
8.  **Concurrency**: Scale out via the process model.
9.  **Disposability**: Maximize robustness with fast startup and graceful shutdown.
10. **Dev/prod parity**: Keep development, staging, and production as similar as possible.
11. **Logs**: Treat logs as event streams.
12. **Admin processes**: Run admin/management tasks as one-off processes.

## Go-specific Context and Examples

### Factor 2: Dependencies (Go Modules)
Go Modules are the standard way to explicitly declare and isolate dependencies.

```bash
# go.mod file
module myapp

go 1.22

require (
    github.com/google/uuid v1.6.0
    github.com/lib/pq v1.10.9
)
```

### Factor 3: Config (Environment Variables)
In Go, you should read configuration from environment variables, not hardcoded files.

```go
package main

import (
	"os"
	"fmt"
)

func main() {
	dbURL := os.Getenv("DATABASE_URL")
	if dbURL == "" {
		dbURL = "postgres://localhost:5432/mydb"
	}
	fmt.Println("Connecting to:", dbURL)
}
```

### Factor 9: Disposability (Graceful Shutdown)
Go makes it easy to handle OS signals for graceful shutdowns, ensuring no data is lost when a process is killed.

```go
package main

import (
	"context"
	"log"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	server := &http.Server{Addr: ":8080"}

	go func() {
		if err := server.ListenAndServe(); err != http.ErrServerClosed {
			log.Fatalf("HTTP server ListenAndServe: %v", err)
		}
	}()

	// Signal handling for graceful shutdown
	stop := make(chan os.Signal, 1)
	signal.Notify(stop, os.Interrupt, syscall.SIGTERM)

	<-stop // Wait for signal

	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
	if err := server.Shutdown(ctx); err != nil {
		log.Fatal("Server Shutdown Failed:", err)
	}
	log.Println("Server gracefully stopped")
}
```

## Interview Questions

**Q: Why should configuration be stored in the environment (Factor 3)?**
**A:** Because it allows the same build (artifact) to be deployed across multiple environments (dev, staging, prod) without modification. It also prevents sensitive information like API keys from being committed to the codebase.

**Q: What does "stateless processes" (Factor 6) mean for application state?**
**A:** It means the application should not store any persistent data in its local memory or filesystem. Any data that needs to persist must be stored in a stateful backing service (like a database or Redis). This allows processes to be killed and restarted on any server without losing information.

**Q: How does Go help with Factor 9 (Disposability)?**
**A:** Go's fast compilation and startup times make it very disposable. Additionally, the standard library provides excellent support for handling system signals, allowing applications to shut down gracefully (finish current requests, close DB connections) before exiting.
