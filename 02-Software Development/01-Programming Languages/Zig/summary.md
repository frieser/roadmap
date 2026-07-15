# Zig — Compact Summary
## Philosophy
- **Zen**: communicate intent precisely. No hidden control flow. No hidden allocations.
- **Explicit**: allocator passed as parameter. No global heap. No GC.
- **Error handling**: `try`/`catch` on error unions (`!T`). Mandatory — compiler rejects unused errors.
- **No preprocessor**: `comptime` replaces macros. Type-safe, debuggable, same language.
- **Compile errors > runtime crashes**: unused vars, type mismatches, overflow caught at compile time.
## Language Basics
- **Arbitrary ints**: `u3`, `i47`, etc. Exact bit-width for registers/bitfields.
- **Alignment**: `@alignOf(T)`, `align(N)`. First-class type property.
- **If/Switch as expressions**: return values. `else` mandatory on `if` used as expression.
- **Payload capture**: `if (opt) |val| {}`, `switch (tagged) { .field => |p| ... }`.
- **For loops**: `for (arr, 0..) |item, idx| {}`. Multi-array simultaneous iteration.
- **Labeled blocks**: `blk: { break :blk value; }`. Block returns value.
- **Functions**: `fn name(params) T`. Params immutable. `inline fn` for forced inlining.
- **Structs**: methods = functions inside struct namespace. `self` param by convention.
- **Tagged unions**: `union(enum) { int: i32, float: f32 }`. Switch exhaustive.
- **Pointers**: `*T` (single, no arithmetic), `[*]T` (many-item, C-like), `[]T` (fat: ptr + len).
## Memory Management
- **Allocator interface**: `std.mem.Allocator`. Explicitly passed. No global default.
- **GPA**: `GeneralPurposeAllocator`. Debug leak detection, use-after-free quarantine. Default for apps.
- **ArenaAllocator**: batch-free. Bulk deallocation. Good for request/phase lifecycles.
- **FixedBufferAllocator**: pre-allocated buffer. Zero syscalls. Fails fast on exhaustion.
- **`defer`**: runs when scope exits. LIFO order. Unconditional cleanup.
- **`errdefer`**: runs ONLY on error exit. Critical for partial-initialization rollback.
- **Safety**: `?T` eliminates null ptrs. Debug bounds checks. `0xAA` fill for `undefined`.
- **`create`/`destroy`** → single item. **`alloc`/`free`** → slices.
## Comptime
- **`comptime`**: execute any Zig code at compile time. Unifies macros, generics, code gen.
- **Generic types**: `fn List(comptime T: type) type {}`. Types are first-class compile-time values.
- **`@typeInfo(T)`**: compile-time reflection. Returns `std.builtin.Type` tagged union.
- **`@Type(...)`**: construct types programmatically from type info descriptors.
- **`@compileError`**: enforce type constraints with custom error messages at compile time.
## Standard Library
- **IO**: `Reader`/`Writer` generic wrappers. `bufferedWriter` for perf. `flush()` required.
- **Strings**: `[]const u8` only. No dedicated `string` type. `std.mem` for ops. `Utf8View` for Unicode.
- **ArrayList**: `std.ArrayList(T).init(allocator)`. `.items` slice. `.toOwnedSlice()` for transfer.
- **HashMap**: `AutoHashMap(K,V)`, `StringHashMap(V)`. Content-hash for string keys.
- **Unmanaged**: `ArrayListUnmanaged` — no stored allocator. Allocator passed to each method.
- **Threads**: `std.Thread.spawn`. 1:1 OS threads. Mutex, Condition, atomics (`@atomicRmw`).
- **No async** in stable. Removed in 0.10. Redesigned "colorless" model targeting 1.0.
## Build System
- **`build.zig`**: Zig program = build graph. Replaces Make/CMake. Full language power.
- **Steps**: `addExecutable`, `addStaticLibrary`, `addSharedLibrary`. `installArtifact`.
- **`build.zig.zon`**: ZON manifest. Decentralized deps by URL + multihash (SHA-256). TOFU.
- **Cross-compilation**: `-Dtarget=x86_64-windows-gnu`. No external toolchains. Bundled libc.
- **`zig cc` / `zig c++`**: drop-in Clang replacements. Cross-compile C/C++ via Zig.
## C Interoperability
- **`@cImport`**: translates C headers at compile time. Single `c.zig` file per project.
- **`@cInclude`** + `linkLibC()`: import + link. `@cDefine` for preprocessor control.
- **`zig translate-c`**: generate permanent Zig bindings. Avoids repeated `@cImport` overhead.
- **`export`**: expose Zig to C. `callconv(.C)`. `extern struct` for ABI struct layout.
- **`emit_h = true`**: auto-generate C header from exported Zig symbols in `build.zig`.
## Tooling & Performance
- **DWARF**: GDB/LLDB. `@breakpoint()` builtin. Mixed C/Zig debugging seamless.
- **Incremental**: `-fincremental`. Binary patching. Sub-100ms rebuilds. Self-hosted backend.
- **Native vs LLVM**: Native (x86/ARM/WASM) = fast dev. LLVM = ReleaseFast/Small optimizations.
- **`test {}` blocks**: inline with code. Access private members. `std.testing.allocator` detects leaks.
- **Fuzzing**: `std.rand.DefaultPrng` for random-input testing. LibFuzzer via C interop.
## Roadmap to 1.0
- **"Colorless" async**: dependency injection of `Io` interface. Same code sync or async.
- **Stability**: stdlib ~85-90% frozen. Strict SemVer post-1.0. `zig fix` for migration tooling.
- **Compiler**: Stage 3 (self-hosted in Zig). Native backends bypass LLVM for debug builds.
- **Target: 2026**. Sub-100ms incremental, cold builds faster than Rust/Go.

| Feature | C | Rust | Zig |
|:---|:---|:---|:---|
| Memory | Manual (malloc) | Ownership/Borrowing | Manual (Allocators) |
| Safety | Unsafe | Verified Safe | Safe defaults (Debug) |
| Generics | `void*` / Macros | Traits / Generics | Comptime |
| Hidden flow | None (mostly) | Destructors | None |
| Cross-compile | Difficult | Moderate | Trivial |

| Mode | Safety | Opt | Backend |
|:---|:---|:---|:---|
| Debug | Yes | No | Native |
| ReleaseSafe | Yes | Yes | LLVM |
| ReleaseFast | No | Max | LLVM |
| ReleaseSmall | No | Size | LLVM |

## Zig Rules
1. No hidden allocations — every alloc passed explicitly.
2. No hidden control flow — `+` never calls a function.
3. No null pointers — `?*T` forces check before dereference.
4. `undefined` is explicit — no accidental uninitialized memory.
5. One `@cImport` per project — single `c.zig` file.
6. `defer` for cleanup, `errdefer` for error-path cleanup.
7. Compile errors beat runtime crashes — catch bugs at build time.
8. `comptime` replaces macros, generics, and code gen — one concept to learn.
