# Go Language — Compact Reference

## Basics
- **Why Go**: static binary, fast compile, goroutine concurrency, GC, single binary deploy.
- **`var` vs `:=`**: `var` for package-level/zero-value explicit; `:=` for function-local inference.
- **Zero values**: `0` numeric, `""` string, `nil` for pointers/slices/maps/chans/interfaces, `false` bool.
- **`const` + `iota`**: compile-time constants; `iota` auto-increments in `const` blocks — enums.
- **Scope**: block-scoped; inner declarations shadow outer; package-level accessible across files.

## Data Types
- **Integers**: `int`/`uint` (platform-dependent), `int8`–`int64`, `uint8`/`byte`, `uint16`, `uint32`/`rune`.
- **Floats**: `float32`, `float64`; `complex64`, `complex128` — no implicit conversions.
- **Strings**: immutable byte sequences; raw literals `` `raw` `` (no escapes) vs interpreted `"escaped\n"`.
- **Runes**: `rune` = `int32`, Unicode code point; `byte` = `uint8`.
- **Type conversion**: explicit only (`T(v)`); no automatic coercion; `strconv` package for string↔number.

## Composite Types
- **Arrays**: fixed size `[N]T`, value semantics (copy on assign); rarely used directly.
- **Slices**: `[]T`, reference to backing array; `len` vs `cap`; `make([]T, len, cap)`; `append` grows.
- **Maps**: `map[K]V`, reference type; comma-ok idiom `v, ok := m[k]`; zero-value `nil` map panics on write.
- **Strings as bytes**: `[]byte` ↔ `string` conversion copies data; `strings`/`bytes` packages.
- **Struct tags**: `` `json:"name"` `` metadata used by `encoding/json`, `gorm`, validation libs.
- **Struct embedding**: composition over inheritance; embedded struct fields/methods promoted.

## Control Flow
- **`for`**: only loop keyword; 3 forms: `for init; cond; post {}`, `for cond {}`, `for {}` (infinite).
- **`for range`**: iterate arrays/slices (`idx, val`), maps (`key, val`), strings (`idx, rune`), channels.
- **`break`/`continue`**: labels supported for multi-level; `goto` exists but discouraged.
- **`if`/`else`**: optional init statement `if v := fn(); v > 0 { ... }`; no parens required.
- **`switch`**: no fallthrough by default; can switch on types (`switch v := x.(type)`) and no condition (alternative to if-else chains).

## Functions
- **Basics**: `func name(params) returnType` or `func name(params) (T, error)`.
- **Multiple returns**: idiomatic for value+error; blank `_` to discard.
- **Variadic**: `func f(args ...T)` → accessed as slice; must be last param.
- **Anonymous/closures**: `func(x int) { ... }`; capture enclosing variables by reference.
- **Named returns**: `func f() (result int)` — naked return returns named vars (OK for short funcs).
- **Call by value**: everything passed by value (copied); use pointers to mutate.

## Pointers
- **Basics**: `*T` type, `&v` address-of, `*p` dereference; no pointer arithmetic (except `unsafe`).
- **With structs**: `(*p).Field` or shorthand `p.Field`; method receivers decide value vs pointer semantics.
- **With maps/slices**: maps/slices already reference types — pointers to them rarely needed.

## Methods & Interfaces
- **Method vs function**: method has receiver `func (r T) M() {}`; can't define on non-local types.
- **Pointer receiver**: can mutate; value receiver gets copy; pointer can call value methods (auto-deref).
- **Value receiver**: safe, immutable view; called on both `T` and `*T`.
- **Interfaces**: implicitly satisfied (no `implements` keyword); `interface{ M1(); M2() }`.
- **Empty interface**: `interface{}` / `any` holds any value; type assertion `v, ok := x.(T)` needed to use.
- **Embedding interfaces**: compose interfaces; `type ReadWriter interface { Reader; Writer }`.
- **Type assertion**: `x.(T)` panics if wrong; comma-ok form safe; type switch `switch v := x.(type)`.

## Generics (Go 1.18+)
- **Type parameters**: `func F[T any](v T)`; `any` = `interface{}`.
- **Generic types**: `type Stack[T any] struct { ... }`; methods on generic type reuse same `T`.
- **Constraints**: interface with type sets: `type Number interface { int | float64 }`.
- **Approximation `~`**: `~int` matches `int` and all types with underlying type `int` (`type MyInt int`).
- **Inference**: compiler deduces `T` from arguments; no partial inference; no return-type-only inference.
- **`comparable`**: built-in constraint for `==`/`!=` and map keys; excludes slices/maps/funcs.

## Error Handling
- **Error interface**: `type error interface { Error() string }`; single-method, implicitly satisfied.
- **`errors.New("msg")`**: simplest error; each call returns unique pointer; used for sentinels.
- **`fmt.Errorf`**: format with `%v` (embed string) or `%w` (wrap preserving original for `errors.Is`/`As`).
- **Sentinel errors**: `var ErrNotFound = errors.New(...)`; checked via `errors.Is(err, ErrNotFound)`.
- **Wrapping chain**: `fmt.Errorf("...: %w", err)` creates `Unwrap()` chain; `errors.Unwrap()` traverses.
- **`errors.Is`**: value-equality check through chain; use instead of `==`.
- **`errors.As`**: type-matching through chain; extracts concrete type fields.
- **Panic/recover**: for unrecoverable bugs, not flow control; `defer func() { if r := recover(); r != nil {...} }()`.

## Concurrency
- **Goroutines**: `go fn()` — lightweight user-space thread (2KB stack, grows); M:N scheduler with work-stealing.
- **Channels**: `make(chan T)` unbuffered (sync handshake) or `make(chan T, n)` buffered (async).
  - **Send**: `ch <- v` (blocks unbuffered till recv, blocks buffered when full).
  - **Receive**: `v := <-ch` (blocks unbuffered till send, blocks buffered when empty).
  - **Close**: `close(ch)` by sender only; receive returns zero-value+false when closed; send on closed = panic.
- **`select`**: wait on multiple channels; random selection if multiple ready; `default` for non-blocking; nil channel blocks forever (disable case).
- **`sync.Mutex`**: `Lock()`/`Unlock()`; `sync.RWMutex` allows concurrent reads; always pass by pointer.
- **`sync.WaitGroup`**: `Add(1)` before goroutine, `defer Done()`, `Wait()` to block; pass by pointer.
- **Worker pool**: N goroutines reading from shared job channel; bounds concurrency, provides backpressure.
- **Context**: `context.WithCancel`/`WithTimeout`/`WithDeadline`; propagate cancellation down call chain; first argument convention; `defer cancel()`.
- **Context values**: custom key types (prevent collision); for request-scoped metadata (traceID, user), not business logic.
- **Patterns**:
  - **Fan-in**: merge N channels into 1 via `WaitGroup` + goroutine per input.
  - **Fan-out**: distribute work from 1 channel to N workers; channels ensure exactly-once delivery.
  - **Pipeline**: chain stages `gen -> sq -> print`; each stage returns channel; closing signals end.
- **Race detector**: `go run -race` / `go test -race`; ~10x memory overhead, finds races that actually occur (not all).

## Testing & Benchmarking
- **`testing` package**: files `*_test.go`; `func TestXxx(t *testing.T)`; `t.Error`/`t.Fatal`/`t.Log`.
- **Table-driven**: define `[]struct{name, input, expected}`, iterate with `t.Run(name, ...)` — idiomatic.
- **Black-box test**: `package foo_test` suffix enforces public-API-only testing.
- **Mocks/stubs**: `gomock`/`mockgen` for interfaces; `httptest.Server` for HTTP handlers.
- **Benchmarks**: `func BenchmarkXxx(b *testing.B)`; `b.N` iterations; `go test -bench=.`.
- **Coverage**: `go test -coverprofile=cover.out && go tool cover -html=cover.out`.
- **Fuzzing**: `func FuzzXxx(f *testing.F)` (Go 1.18+); random inputs for edge-case discovery.

## Standard Library Highlights
- **I/O**: `io.Reader`/`io.Writer` interfaces; `os.File`; `bufio` buffered I/O; `io.ReadAll`.
- **JSON**: `encoding/json`; `Marshal`/`Unmarshal`; struct tags; `json.Encoder`/`Decoder` for streams.
- **Net/HTTP**: `http.HandleFunc`, `http.ListenAndServe`; `http.Handler` interface; middleware via wrapping.
- **Context**: `context.Context` for deadlines, cancellation, request-scoped values.
- **`embed`**: `//go:embed` compiles static files into binary; `embed.FS` for directories.
- **Time/flag**: `time.Now`, `time.Duration`, `time.Ticker`; `flag` for CLI arg parsing.
- **Slog**: structured logging (Go 1.21+); `slog.Info("msg", "key", val)`; JSON output.

## Ecosystem
- **CLI**: Cobra (cmd/ structure, large projects), `urfave/cli` (declarative, small tools), Bubbletea (TUI, Elm Architecture).
- **Web frameworks**: Gin (fast, radix-tree routing), Echo (minimalist), Fiber (fasthttp, zero-alloc), Beego (MVC full-stack), Gorilla Mux (classic, archived).
- **HTTP routing**: Go 1.22+ `ServeMux` supports `GET /path`, `/{id}` wildcards — often eliminates need for frameworks.
- **gRPC**: protobuf IDL + codegen; HTTP/2; interceptors; strict typing.
- **DB**: `pgx` (Postgres, binary protocol, pooling), GORM (ORM, auto-migration, preloading, hooks).
- **Logging**: Zerolog (zero-allocation, JSON), Zap (Uber, fast, Logger vs SugaredLogger).
- **Realtime**: Melody (WebSocket wrapper), Centrifugo (standalone scalable broker, JWT auth, presence).

## Go Toolchain
- **`go run`**: compile+run; `go build -o bin` static binary; `go install` installs to `$GOPATH/bin`.
- **`go fmt`**: canonical formatting; `goimports` adds/removes imports automatically.
- **`go mod`**: dependency management; `go mod tidy` prunes; `go.work` for multi-module workspaces.
- **`go vet`**: static analysis; `staticcheck`/`golangci-lint` for deeper checks; `govulncheck` call-graph security.
- **`go generate`**: runs `//go:generate ...` directives; for codegen (stringer, mockgen, protobuf); commit generated files.
- **`go test`**: `-race`, `-cover`, `-bench`, `-run`, `-count`.
- **Build tags**: `//go:build linux && amd64` or file suffixes `_linux.go`; `-tags integration` for conditional tests.
- **Cross-compile**: `GOOS=linux GOARCH=arm64 go build`; `CGO_ENABLED=0` for pure static.
- **Compile flags**: `-ldflags "-s -w"` strip debug info; `-ldflags "-X main.Version=1.0"` inject vars; `-trimpath` for reproducibility.

## Performance, Debugging, Memory
- **pprof**: `net/http/pprof` for CPU/heap/goroutine/block profiles; `go tool pprof -http=:8081` for flame graphs.
- **Trace**: `go tool trace` for goroutine scheduling, GC pauses, latency diagnosis.
- **Memory**: stack (fast, auto-cleanup), heap (GC-managed); TCMalloc-inspired: `mcache`→`mcentral`→`mheap`.
- **Escape analysis**: compiler decides stack vs heap; `go build -gcflags '-m'` shows escapes; returning pointer → heap.
- **GC**: non-generational, concurrent, tricolor mark-and-sweep; `GOGC` env var controls frequency (default 100).
- **Reflection**: `reflect.TypeOf`/`ValueOf`; slow, runtime panics instead of compiler errors; use sparingly.
- **`unsafe`**: `unsafe.Pointer` bypasses type safety; `uintptr` for pointer arithmetic; GC does not track uintptr.
- **CGO**: `import "C"` calls C code; disables cross-compile; manual memory management for C allocations; stack-switching overhead.

## Go Rules
1. Share memory by communicating, not communicate by sharing memory.
2. Errors are values — check `if err != nil` explicitly.
3. Pass `context.Context` as first argument; never store in structs.
4. Always `defer cancel()` after creating a context with timeout/cancel.
5. Always close HTTP response bodies (`defer resp.Body.Close()`).
6. Pass mutexes and waitgroups by pointer; never copy after first use.
7. Don't use `panic` for expected errors; return `error` instead.
8. Commit generated code (`go generate`); don't require generators at build time.
9. Use table-driven tests with `t.Run()` for subtests.
10. Run `go test -race` in CI; the race detector finds real bugs.
