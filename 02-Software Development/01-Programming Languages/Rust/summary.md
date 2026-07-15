---
---

## Rust — Compressed Reference

### WHAT IS RUST
- **Systems language**: zero-cost abstractions, no GC, compile-time safety guarantees.
- **Core innovation**: Ownership system — memory + thread safety without runtime overhead.
- **Compared to C++**: safe by default, modern tooling (Cargo). Compared to Go: no GC, steeper curve, faster runtime.

### LANGUAGE BASICS
- **Immutability default**: `let` binds; `let mut` allows mutation. Shadowing allows type transformations.
- **Scalar types**: `i32` (default), `u8..u128`, `f64` (default), `bool`, `char` (4-byte Unicode).
- **Compound types**: Tuple `(T1, T2)` — fixed size, mixed types. Array `[T; N]` — fixed size, same type, stack-allocated.
- **Expression-based**: `if` returns values (no ternary). `loop` returns via `break val`. Semicolon suppresses return.
- **Control flow**: `for` over iterators (safe, no bounds bugs). `while` / `loop` for custom logic. Labels for nested breaks.
- **Pattern matching**: `match` exhaustive, binds data from variants. `if let` for single-pattern convenience. Destructuring on structs/enums/tuples.
- **Functions**: `fn name(args) -> RetType`. Last expression = implicit return. `&self`/`&mut self`/`self` in methods.
- **Enums**: Algebraic data types — each variant can hold data. `Option<T>` = `Some(T) | None` (null replacement). `Result<T, E>` = `Ok(T) | Err(E)`.
- **Structs**: Classic (`name: Type`), tuple (`Point(i32, i32)`), unit (`struct AlwaysEqual;`). Field init shorthand, struct update syntax `..base`.
- **Impl blocks**: Separate data from behavior. Methods vs associated functions (`::new()`). Multiple impl blocks allowed.

### DATA STRUCTURES (stdlib)
- **Vec\<T\>**: Heap-allocated growable array. Used as stack (`push`/`pop` O(1)). Default collection.
- **VecDeque\<T\>**: Ring-buffer double-ended queue. `push_back`/`pop_front` O(1). FIFO queue of choice.
- **HashMap\<K,V\>**: Unordered O(1) lookups. Key requires `Eq + Hash`. Entry API for idiomatic updates.
- **HashSet\<T\>**: Unique set = `HashMap<T, ()>`. Fast membership. Union/intersection via iterators.
- **BTreeMap / BTreeSet**: Sorted, O(log N). Range queries. Cache-friendly B-Tree over RB-Tree.
- **BinaryHeap**: Max-heap by default. Min-heap via `Reverse<T>`. O(log N) push/pop.
- **LinkedList**: Doubly-linked. Almost never used — `Vec`/`VecDeque` preferred (cache locality).
- **String**: Owned, growable UTF-8 buffer. `&str`: borrowed slice. No integer indexing (multi-byte chars).

### OWNERSHIP (10 lines)
- **Three rules**: (1) Each value has exactly one owner. (2) Only one owner at a time. (3) Drop on scope exit.
- **Move semantics**: Assigning non-`Copy` type transfers ownership. Original invalidated. Prevents double-free.
- **Copy trait**: Stack-only types (`i32`, `bool`, `char`) bitwise copy on assignment. No ownership transfer.
- **Clone**: Explicit deep copy with `.clone()`. Both original and clone valid (heap duplication).
- **Borrowing**: `&T` — shared read-only reference. `&mut T` — exclusive mutable reference.
- **Borrow checker rule**: One `&mut T` OR any number of `&T`. Prevents data races at compile time.
- **Dangling prevention**: Cannot return reference to locally created value. Compiler tracks lifetime scopes.
- **Slices**: `&str` and `&[T]` are fat pointers (ptr + len). Views into String/Vec without copying.
- **Stack vs Heap**: Stack = fast, fixed-size, LIFO. Heap = dynamic, pointer-based. `Box<T>` forces heap allocation.
- **Drop**: Automatic destructor call at scope end. RAII pattern — no manual `free`.

### LIFETIMES (5 lines)
- **Goal**: Ensure references never outlive referent. Annotations (`'a`) describe relationships, don't extend lifetime.
- **Explicit annotations**: `fn foo<'a>(x: &'a str, y: &'a str) -> &'a str`. Structs with refs need `struct Foo<'a> { r: &'a T }`.
- **Elision rules**: (1) Each input ref gets its own lifetime. (2) 1 input → output gets same lifetime. (3) `&self` → output tied to self.
- **Variance**: `&'a T` covariant (longer OK for shorter). `&'a mut T` invariant over T (safety guard). `'static` is subtype of all.
- **`'static`**: Valid for entire program. String literals = `&'static str`. Use sparingly — not for owned data.

### TRAITS & GENERICS (5 lines)
- **Trait definition**: `trait Foo { fn bar(&self) -> T; }`. Default implementations supported. Orphan rule: impl trait OR type must be local.
- **Trait bounds**: `fn foo<T: Display + Clone>(t: T)`. `where` clauses for readability. Supertraits: `trait B: A { }`.
- **Static dispatch**: `impl Trait` / generics → monomorphization (specialized copy per type). Zero-cost, larger binary.
- **Dynamic dispatch**: `dyn Trait` → vtable lookup at runtime. Enables heterogeneous collections (`Vec<Box<dyn Trait>>`).
- **Associated types**: `type Item;` in trait. One-to-one relationship. Vs generics for many-to-many. `?Sized` relaxes size constraint.

### ERROR HANDLING
- **No exceptions**: `Option<T>` for absence, `Result<T, E>` for failure. Compiler warns on ignored Results.
- **`?` operator**: Propagates `Err` upward via `From` conversion. `main()` can return `Result`.
- **Custom errors**: `thiserror` for libraries (derive `Error`). `anyhow` for applications (opaque error handling).

### TESTING
- **Unit tests**: `#[cfg(test)] mod tests` in same file. Access private items via `super::*`.
- **Integration tests**: `tests/` directory, separate crate, only public API.
- **Doc tests**: `/// ```rust` code blocks compile + run via `cargo test`.
- **Mocking**: `mockall` crate — `#[automock]` on traits + dependency injection.
- **Property-based**: `proptest` — test invariants over random inputs with automatic shrinking.

### MODULES & CRATES
- **Module system**: `mod` declares. Privacy: private by default, `pub`/`pub(crate)`/`pub(super)`. `use` brings paths in.
- **Cargo**: `Cargo.toml` = config (SemVer: `^1.2.3`). `Cargo.lock` = pin (commit for binaries). Workspaces = multi-crate repos.
- **crates.io**: Publish via `cargo publish`. Code immutable; `cargo yank` to deprecate. `docs.rs` auto-builds docs.

### CONCURRENCY & PARALLELISM (8 lines)
- **Threads**: `std::thread::spawn` — 1:1 OS threads. `move` closure transfers ownership. `.join()` waits.
- **Channels**: `mpsc::channel()` — multi-producer, single-consumer. `send()`/`recv()`. Drop sender → receiver terminates.
- **Send/Sync**: Marker traits. `Send` = ownership can cross threads. `Sync` = `&T` safe across threads. `Rc` is neither; `Arc` is both.
- **Arc\<T\>**: Atomic reference counting. Shared immutable access across threads. Not `Copy` — explicit `.clone()`.
- **Mutex\<T\>**: Exclusive mutable access. Lock returns `MutexGuard` (RAII unlock on drop). Poisoned on panic.
- **RwLock\<T\>**: Multiple readers OR one writer. Read-heavy workloads only. No lock upgrade.
- **Atomics**: `AtomicUsize` etc. Non-blocking CPU-level ops. Memory ordering: `Relaxed` (counter) → `Acquire/Release` (publish) → `SeqCst` (global).
- **Async/await**: Cooperative multitasking. `Future` trait = `poll()`. Tokio = dominant runtime (work-stealing). `Pin` prevents self-referential moves.

### MACROS (5 lines)
- **Declarative**: `macro_rules!` — pattern matching over tokens. Fragment specifiers: `$expr`, `$ident`, `$ty`, `$tt`. Repeat: `$(...),*`.
- **Procedural**: Compiler plugins. Types: custom derive (`#[derive]`), attribute-like, function-like. Must live in separate `proc-macro` crate.
- **`syn` + `quote`**: `syn` parses `TokenStream` → AST. `quote` generates code via quasi-quoting. Standard proc-macro toolkit.
- **Hygiene**: Declarative macros are partially hygienic (local vars safe, items leak). Proc-macros are not hygienic — use fully qualified paths.
- **DSLs**: Macros enable internal DSLs (e.g., `html!`, `sqlx::query!`). Compile-time validation vs builder-pattern tradeoff.

### ECOSYSTEM (Key Crates)
- **Tokio**: Async runtime. Work-stealing scheduler, non-blocking I/O, timers. Foundation for Axum, Reqwest, SQLx.
- **Axum**: Web framework. Tower-based, extractor pattern, macro-free. Hyper underneath. Type-safe routing.
- **Reqwest**: HTTP client. Connection pooling, JSON auto, async + blocking. Wraps Hyper.
- **Serde**: Serialization framework. `#[derive(Serialize, Deserialize)]`. Format-agnostic. `serde_json` for JSON.
- **SQLx / Diesel**: Async (SQLx, compile-time checked) vs sync (Diesel, type-safe query builder) ORMs.
- **Clap**: CLI argument parser. Derive-based, auto-generated help. `structopt` predecessor.
- **Criterion**: Microbenchmarking. Statistical analysis, regression detection, HTML reports. Works on stable.
- **WASM**: `wasm-pack` + `wasm-bindgen`. First-class Rust → WebAssembly support.

### RUST RULES
1. If it compiles, no data races. No null pointers. No double frees.
2. `cargo clippy` before committing. `cargo fmt` always.
3. Prefer `&str` over `&String` in function params. Prefer `impl Trait` over `dyn Trait` when possible.
4. Never `unwrap()` in production. Use `?`, `expect()` with reason, or proper `match`.
5. `Vec` over `LinkedList`. `VecDeque` over `Vec` for queues.
6. `Arc<Mutex<T>>` for shared mutable state across threads. `Arc<RwLock<T>>` if read-heavy.
7. Don't block inside async. Use `tokio::spawn_blocking` for CPU work.
8. Derive `Debug`, `Clone`, `PartialEq` by default. Document public API with `///`.
9. One `&mut T` exists → all other borrows blocked. Reborrow with nested scopes.
10. Trust the borrow checker. Fighting it usually means wrong design, not compiler bug.
