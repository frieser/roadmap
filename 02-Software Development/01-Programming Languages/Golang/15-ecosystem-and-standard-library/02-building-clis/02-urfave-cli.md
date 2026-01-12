# urfave/cli

## Summary
`urfave/cli` is a simple, fast, and fun package for building command line apps in Go. It is less "opinionated" and structured than Cobra, making it ideal for smaller tools, developer scripts, or applications where you want everything in a single `main.go` file rather than a dispersed `cmd/` directory structure.

## Detailed Explanation

### Philosophy
Unlike Cobra, which encourages a directory-based structure, `urfave/cli` allows you to define your entire application tree (commands, flags, actions) declaratively inside your `main` function.

### Basic Structure
The app is defined as a `&cli.App` struct.

```go
package main

import (
	"fmt"
	"log"
	"os"

	"github.com/urfave/cli/v2"
)

func main() {
	app := &cli.App{
		Name:  "greet",
		Usage: "fight the loneliness!",
		
		// Global Flags
		Flags: []cli.Flag{
			&cli.StringFlag{
				Name:    "lang",
				Aliases: []string{"l"},
				Value:   "english",
				Usage:   "language for the greeting",
			},
		},
		
		// Commands
		Commands: []*cli.Command{
			{
				Name:    "hello",
				Aliases: []string{"hi"},
				Usage:   "say hello to someone",
				Action: func(c *cli.Context) error {
					name := "World"
					if c.NArg() > 0 {
						name = c.Args().Get(0)
					}
					
					if c.String("lang") == "spanish" {
						fmt.Println("Hola", name)
					} else {
						fmt.Println("Hello", name)
					}
					return nil
				},
			},
		},
	}

	if err := app.Run(os.Args); err != nil {
		log.Fatal(err)
	}
}
```

### Key Differences vs Cobra
| Feature | Cobra | urfave/cli |
| :--- | :--- | :--- |
| **Structure** | `cmd/` folder, one file per command | Declarative struct, often single file |
| **Flags** | POSIX-compliant (pflag) | Custom flag parsing |
| **Context** | Uses `cmd *cobra.Command` | Uses `c *cli.Context` |
| **Use Case** | Large, complex CLIs (Kubernetes) | Simple tools, dev scripts |

### Features
*   **Aliases**: Easy support for command aliases (`hi` -> `hello`).
*   **Bash Completion**: Built-in support.
*   **Version Printing**: Automatic `-v` / `--version` support.
*   **Categories**: Group commands in help output.

## Interview Questions

**Q: When would you choose `urfave/cli` over `Cobra`?**
**A:** `urfave/cli` is often better for smaller, self-contained utilities where the overhead of Cobra's directory structure feels excessive. Its declarative style makes it very readable for simple apps defined in a single file. Cobra is preferred for large, multi-subcommand applications that need enterprise-grade structure and extensive plugin ecosystems.

**Q: How do you access flag values in an `urfave/cli` action?**
**A:** You access them via the `cli.Context` passed to the Action function. For example, `c.String("myflag")`, `c.Int("count")`, or `c.Bool("verbose")`. This context also provides access to arguments (`c.Args()`).

**Q: Does `urfave/cli` support subcommands?**
**A:** Yes. The `cli.Command` struct has a `Subcommands` field, which is a slice of `[]*cli.Command`. This allows you to nest commands recursively (e.g., `app user create`).
