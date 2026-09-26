# Version Changelog

## Summary
A chronological log of major changes in the Go language and toolchain, focusing on features that impact daily development (Generics, Workspaces, Iterators, Generic Methods) and critical performance improvements (PGO, Green Tea GC, Memory Model).

## Detailed Explanation

### Go 1.18 (March 2022) - The Generics Era
*   **Generics**: Added type parameters (`[T any]`) to functions and types. Introduction of the `any` keyword (alias for `interface{}`).
*   **Fuzzing**: Built-in support for fuzz testing (`testing.F`) to find edge cases automatically.
*   **Workspaces**: Multi-module development with `go.work` files, allowing seamless local replacements of dependencies.

### Go 1.19 (August 2022)
*   **Memory Model**: Formalized memory model for atomic operations, aligning with C++20. Added `atomic.Pointer[T]`.
*   **Doc Comments**: Support for links, lists, and headings in doc comments.
*   **Soft Memory Limit**: `GOMEMLIMIT` environment variable to better tune GC in containerized environments.

### Go 1.20 (February 2023)
*   **Profile-Guided Optimization (PGO)**: Preview support. The compiler can use a CPU profile (`default.pgo`) to optimize hot paths (inline functions, devirtualize calls).
*   **Multierrors**: `errors.Join` supports wrapping multiple errors.
*   **Comparability**: `comparable` constraint is now satisfied by interface types even if the underlying type isn't comparable (runtime panic instead of compile error).

### Go 1.21 (August 2023)
*   **Built-ins**: Added `min`, `max`, and `clear` (for maps/slices).
*   **Standard Library**:
    *   `log/slog`: Structured logging.
    *   `slices` and `maps`: Generic utility functions (`slices.Sort`, `maps.Clone`).
    *   `cmp`: Comparability utilities.
*   **PGO**: Now General Availability (GA).
*   **Loop Variable Preview**: `GOEXPERIMENT=loopvar` introduced to fix loop variable capture semantics.

### Go 1.22 (February 2024)
*   **Loop Variable Fix**: Loop variables are now scoped per iteration, not per loop. No more `v := v` inside loops!
*   **Range Over Integers**: `for i := range 10` is now valid.
*   **math/rand/v2**: Faster, safer, and cleaner random number generation.
*   **HTTP Routing**: `net/http.ServeMux` now supports methods (`GET /path`) and wildcards (`/items/{id}`).

### Go 1.23 (August 2024)
*   **Iterators**: Standardized iterators via `iter` package (`Seq`, `Seq2`).
*   **Range Over Func**: The `range` clause now supports iterator functions (`func(func() bool)`).
*   **Unique**: New `unique` package for canonicalizing comparable values (interning strings/structs).
*   **Timer/Ticker Changes**: `time.Timer` and `time.Ticker` channels are now unbuffered, and unstopped timers are collectable by the GC.

### Go 1.24 (February 2025)
*   **Generic Type Aliases**: Full support for parameterized type aliases (`type Set[T comparable] = map[T]struct{}`).
*   **Weak Pointers**: New `weak` package for weak references (GC can collect the object even if the weak pointer exists).
*   **Tool Dependencies**: `tool` directives in `go.mod` track executable dependencies, replacing the old `tools.go` workaround.
*   **Map Implementation**: Switched to Swiss Tables, improving map performance.
*   **JSON v2 Preview**: `encoding/json/v2` available behind `GOEXPERIMENT=jsonv2`.
*   **Build Info**: `go build` embeds VCS tag/commit version into the binary (`+dirty` suffix on uncommitted changes).
*   **GOAUTH**: New environment variable for flexible authentication of private module fetches.

### Go 1.25 (August 2025)
*   **Container-Aware `GOMAXPROCS`**: On Linux the runtime now respects the cgroup CPU bandwidth limit (Kubernetes "CPU limit"), and updates `GOMAXPROCS` dynamically when the available CPUs or limits change.
*   **New Garbage Collector (experimental)**: The "Green Tea" GC (`GOEXPERIMENT=greenteagc`) reduces GC overhead 10–40% on GC-heavy workloads via better marking/scaning of small objects.
*   **Trace Flight Recorder**: `runtime/trace.FlightRecorder` records execution traces into an in-memory ring buffer, snapshotting the last few seconds on demand — cheap enough for production debugging of rare events.
*   **`go.mod` `ignore` Directive**: Specify directories the `go` command should ignore when matching package patterns.
*   **New Vet Analyzers**: `waitgroup` (misplaced `sync.WaitGroup.Add`) and `hostport` (IPv6-unsafe `fmt.Sprintf("%s:%d", ...)` addresses).
*   **Testing**: `testing/synctest` promoted to stable for deterministic concurrent testing.
*   **`go doc -http`**: Starts a documentation server and opens it in a browser. `go version -m -json` prints embedded `BuildInfo` as JSON.

### Go 1.26 (February 2026)
*   **`new` with Expressions**: The built-in `new` now accepts an expression operand to initialize the variable — e.g. `new(yearsSince(born))` for optional pointer fields.
*   **Self-Referential Generic Constraints**: A generic type may now refer to itself in its own type parameter list (`type Adder[A Adder[A]] interface{ ... }`).
*   **Green Tea GC Enabled by Default**: The new collector shipped as experimental in 1.25 is now the default (disable via `GOEXPERIMENT=nogreenteagc`, expected removed in 1.27).
*   **Revamped `go fix`**: Now home to Go's *modernizers* — push-button fixes for modern idioms/APIs, plus a source-level inliner driven by `//go:fix inline` directives.
*   **`go mod init` Default Version**: New modules default to `go 1.(N-1).0` to encourage compatibility with currently supported Go versions.
*   **Faster cgo**: Baseline cgo call overhead reduced by ~30%.
*   **Heap Base Address Randomization**: 64-bit runtimes randomize the heap base address, hardening cgo programs against memory-address prediction.
*   **Goroutine Leak Profile (experimental)**: New `goroutineleak` pprof profile (`GOEXPERIMENT=goroutineleakprofile`) surfaces leaked goroutines.
*   **Pprof UI**: `pprof -http` now defaults to the flame graph view.

### Go 1.27 (August 2026)
*   **Generic Methods**: A method declaration may now declare its own type parameters — generic behavior scoped to a type's namespace instead of a package-level function. Example: `(*Rand) N[Int intType](Int) Int` in `math/rand/v2`.
*   **Generalized Function Type Inference**: Type inference now applies wherever a generic function is assigned to (or converted to) a matching function type.
*   **Struct Literal Keys**: A key in a struct literal may be any valid field selector for the struct type, not just a top-level field name.
*   **`go test` `stdversion` Check**: `go test` now runs the `stdversion` vet check by default, flagging stdlib symbols too new for the module's `go` version.
*   **`go doc` Improvements**: Supports `package@version` syntax and an `-ex` option to list/print executable examples.
*   **`go mod tidy`**: For modules on `go 1.27`, duplicate `require` blocks are automatically merged into the standard two-block layout (direct + indirect), preserving attached comments.
*   **More `go fix` Modernizers**: Adds `atomictypes`, `embedlit`, `slicesbackward`, and `unsafefuncs`.
*   **`GODEBUG` Compatibility**: The `go` command now accepts removed `GODEBUG` settings (in `go.mod`/`//go:debug`) if set to their final default value, honoring the Go 1 compatibility promise.
*   **Tooling**: Response files (`@file`) supported by `compile`, `link`, `asm`, `cgo`, `cover`, and `pack`. `bzr` version control support dropped.

### Code Example: Evolution of Loops and Generics

```go
package main

import (
    "fmt"
    "slices"
)

// Go 1.18+: Generics
func Reverse[T any](s []T) {
    for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
        s[i], s[j] = s[j], s[i]
    }
}

// Go 1.25+: Generic methods (methods with their own type parameters)
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(v T) {
    s.items = append(s.items, v)
}

func (s *Stack[T]) Map[U any](f func(T) U) []U {
    out := make([]U, len(s.items))
    for i, v := range s.items {
        out[i] = f(v)
    }
    return out
}

func main() {
    nums := []int{1, 2, 3}
    Reverse(nums)

    // Go 1.21+: slices package
    slices.Sort(nums)

    // Go 1.22+: Loop variable scoping (safe to capture pointers)
    var ptrs []*int
    for _, v := range nums {
        // Pre-1.22: This would capture the same address
        // 1.22+: Captures the value of the current iteration
        ptrs = append(ptrs, &v)
    }

    // Go 1.22+: Range over integers
    for i := range 5 {
        fmt.Println(i) // 0, 1, 2, 3, 4
    }

    // Go 1.26+: new() with an initial value expression
    p := new(42) // *int pointing to 42
    fmt.Println(*p)
}
```

## Interview Questions

**Q: What changed with loop variables in Go 1.22?**
**A:** Before 1.22, loop variables were reused across iterations, causing bugs when capturing their address or using them in closures (goroutines). In 1.22, the variable is effectively redeclared for each iteration, fixing this common "gotcha".

**Q: What is Profile-Guided Optimization (PGO) in Go?**
**A:** PGO allows the compiler to use a runtime profile (CPU pprof) from a previous run to make optimization decisions, such as which functions to inline. It typically yields a 2-10% performance improvement.

**Q: What is the purpose of the `any` keyword introduced in Go 1.18?**
**A:** `any` is a type alias for `interface{}`. It was introduced alongside Generics to make function signatures with type parameters (like `[T any]`) more readable.

**Q: What is the Green Tea garbage collector?**
**A:** Introduced as an experiment in Go 1.25 (`GOEXPERIMENT=greenteagc`) and enabled by default in Go 1.26, it improves the marking and scanning of small objects through better locality and CPU scalability, reducing GC overhead by roughly 10-40% in GC-heavy programs. It can be disabled with `GOEXPERIMENT=nogreenteagc` (opt-out expected to be removed in Go 1.27).

**Q: What are generic methods introduced in Go 1.27?**
**A:** Generic methods allow a method declaration to declare its own type parameters, e.g. `func (r *Rand) N[Int intType](n Int) Int`. This lets you add generic behavior within a type's namespace instead of declaring package-level generic functions. Note: interface methods may not declare type parameters, and interface methods cannot be implemented by generic methods.

**Q: How does Go 1.25 change `GOMAXPROCS` in containers?**
**A:** On Linux, `GOMAXPROCS` now defaults to the lower of the available logical CPUs and the cgroup CPU bandwidth limit (Kubernetes "CPU limit"), and it is updated dynamically when either changes. It does not consider "CPU requests". This behavior is disabled if `GOMAXPROCS` is set manually, or via `GODEBUG=containermaxprocs=0` / `updatemaxprocs=0`.
