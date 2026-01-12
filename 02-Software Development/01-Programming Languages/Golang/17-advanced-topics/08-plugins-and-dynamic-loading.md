# Plugins and Dynamic Loading

## Summary
Go is statically linked by default, but the `plugin` package allows loading shared object files (`.so`) at runtime. However, this mechanism is fragile and platform-limited. Most Go architectures prefer **RPC-based plugins** (over gRPC or net/rpc) for better isolation and compatibility.

## Detailed Explanation

### The `plugin` Package
*   **Capabilities**: Load a `.so` file, lookup exported symbols (functions/variables), and use them.
*   **Constraints (The "Why not" list)**:
    1.  **Platform Support**: Works mainly on Linux and macOS. Windows support is poor/non-existent.
    2.  **Versioning Hell**: The plugin and the host app must be compiled with the **exact same Go version** and **exact same dependency versions**. If `mod.sum` differs even slightly, it fails.
    3.  **Shared Memory**: A plugin crash crashes the host.

### Alternatives: RPC / HashiCorp go-plugin
The industry standard for Go plugins is to run the plugin as a separate **subprocess** and communicate via RPC (gRPC).
*   **Pros**: Crash isolation, can be written in any language, no dependency version coupling.
*   **Cons**: Slower (IPC overhead) compared to direct function calls.

### Code Example: Native Plugin (Fragile)

**plugin.go**
```go
package main

import "fmt"

func Greeter(name string) {
    fmt.Printf("Hello %s from plugin!\n", name)
}
```
Build: `go build -buildmode=plugin -o myplugin.so plugin.go`

**main.go**
```go
package main

import (
    "plugin"
)

func main() {
    p, err := plugin.Open("myplugin.so")
    if err != nil {
        panic(err)
    }

    sym, err := p.Lookup("Greeter")
    if err != nil {
        panic(err)
    }

    // Cast to expected function signature
    greeterFunc, ok := sym.(func(string))
    if !ok {
        panic("Plugin has wrong signature")
    }

    greeterFunc("World")
}
```

## Interview Questions

**Q: Can Go load code dynamically?**
**A:** Yes, via the `plugin` package, but it loads compiled shared objects, not source code.

**Q: Why is the `plugin` package often discouraged?**
**A:** It requires the host and plugin to have identical build environments (Go version, dependencies), making it very brittle. It also lacks process isolation.

**Q: What is the preferred alternative for a plugin system in Go?**
**A:** Running plugins as separate processes communicating via gRPC (like HashiCorp's `go-plugin` system). This decouples build requirements and isolates crashes.
