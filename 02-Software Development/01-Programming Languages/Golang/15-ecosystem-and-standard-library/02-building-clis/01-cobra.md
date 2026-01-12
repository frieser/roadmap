# Cobra

## Summary
Cobra is the most popular library for creating modern CLI applications in Go. It powers massive projects like Kubernetes, Hugo, and GitHub CLI. It is built on the philosophy of Commands, Arguments, and Flags, and provides features like automatic help generation, subcommand nesting, and shell auto-completion out of the box.

## Detailed Explanation

### Core Concepts
Cobra organizes CLIs into a structure of `APPNAME COMMAND ARG --FLAG`.
*   **Commands**: The actions (e.g., `git commit`, `git push`).
*   **Args**: The things being acted upon.
*   **Flags**: Modifiers for the action (e.g., `-f`, `--verbose`).

### Setup and Structure
The standard Cobra application structure is:
```
/
├── cmd/
│   ├── root.go       # The base command (entry point)
│   ├── serve.go      # A subcommand
│   └── config.go     # Another subcommand
└── main.go           # Simply calls cmd.Execute()
```

#### 1. The Root Command (`root.go`)
This command represents the binary itself (e.g., `hugo`).
```go
package cmd

import (
	"fmt"
	"os"
	"github.com/spf13/cobra"
)

var rootCmd = &cobra.Command{
	Use:   "myapp",
	Short: "A brief description of your application",
	Long:  `A longer description...`,
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("Hello from root command")
	},
}

func Execute() {
	if err := rootCmd.Execute(); err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
}
```

#### 2. Adding Subcommands (`serve.go`)
```go
package cmd

import (
	"fmt"
	"github.com/spf13/cobra"
)

var port string

// serveCmd represents the serve command
var serveCmd = &cobra.Command{
	Use:   "serve",
	Short: "Start the server",
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Printf("Server starting on port %s...\n", port)
	},
}

func init() {
	rootCmd.AddCommand(serveCmd)

	// Local Flag: Only available to this command
	serveCmd.Flags().StringVarP(&port, "port", "p", "8080", "Port to run on")
}
```

### Key Features
1.  **Persistent Flags**: Flags defined on a parent command that are available to all children (e.g., `--config` or `--verbose` defined on root).
    ```go
    rootCmd.PersistentFlags().BoolVarP(&Verbose, "verbose", "v", false, "verbose output")
    ```
2.  **Hooks**: `PersistentPreRun`, `PreRun`, `PostRun` allow you to run code before/after the command execution logic.
3.  **Viper Integration**: Cobra works seamlessly with **Viper** for configuration management, allowing flags to be bound to config file values or environment variables.

## Interview Questions

**Q: What is the difference between `Flags()` and `PersistentFlags()` in Cobra?**
**A:** `Flags()` are local to the specific command they are assigned to. `PersistentFlags()` are inherited by the command *and all of its subcommands*. For example, a global `--verbose` flag should be a persistent flag on the root command, while a `--port` flag for a specific `server` command should be local.

**Q: How does Cobra handle command hierarchy execution order?**
**A:** Cobra executes hooks in this order:
1.  `PersistentPreRun` (parents first)
2.  `PreRun`
3.  `Run` (The main logic)
4.  `PostRun`
5.  `PersistentPostRun`
If any `PreRun` returns an error, the execution stops.

**Q: Why is `cobra-cli` (the generator tool) recommended for starting projects?**
**A:** It bootstraps the project with the correct directory structure (`cmd/`) and boilerplate code (`root.go`), saving time and ensuring best practices are followed. It allows you to add commands easily (`cobra-cli add user`) without manually wiring up the `init()` functions.
