# rust-gdb
---
---

## Summary
`rust-gdb` is a wrapper script bundled with the Rust toolchain that launches the standard GNU Debugger (GDB) configured specifically for Rust. It automatically loads Python "pretty-printers" that visualize Rust's complex standard types (like `Vec`, `String`, `Option`, and Enums) in a human-readable format, making debugging significantly easier than using raw GDB.

## Detailed Explanation

### What is it?
It is essentially a shell script that sets up the environment before calling the actual `gdb` binary. Without this wrapper, inspecting a simple `String` in GDB would reveal its raw internal structure (a pointer, capacity, and length), which is tedious to interpret. `rust-gdb` interprets these structures and displays the actual string content.

### Difference from Standard `gdb`
1.  **Pretty-Printers**: The main feature. It transforms `struct { ptr: 0x..., cap: 5, len: 5 }` into `Vec(vec![1, 2, 3, 4, 5])`.
2.  **Symbol Demangling**: Rust compiler "mangles" function names (e.g., `_ZN3std3fmt...`). `rust-gdb` helps in displaying them as readable Rust paths like `std::fmt::...`.
3.  **Auto-Configuration**: It avoids the need to manually source the Rust GDB scripts in your `.gdbinit` file.

### Usage Workflow

1.  **Compile**: Ensure your project is compiled with debug symbols (default in `dev` profile).
    ```bash
    cargo build
    ```
    *Note: If debugging a release build, add `debug = true` to `[profile.release]` in `Cargo.toml`.*

2.  **Start Debugger**:
    ```bash
    rust-gdb target/debug/my_project
    ```

3.  **Interactive Session**:
    ```text
    (gdb) break main.rs:10
    (gdb) run
    (gdb) print my_variable
    ```

### Key Commands (Cheatsheet)
| Command | Abbrev | Description |
| :--- | :--- | :--- |
| `break [file]:[line]` | `b` | Set a breakpoint at a specific line. |
| `run [args]` | `r` | Start the program with optional arguments. |
| `next` | `n` | Step **over** the next line of code. |
| `step` | `s` | Step **into** the function call. |
| `print [var]` | `p` | Print the value of a variable. |
| `backtrace` | `bt` | Show the current call stack. |
| `continue` | `c` | Continue execution until the next breakpoint. |
| `info locals` | | Show all local variables in the current frame. |

## Interview Questions

1.  **Q: Why use `rust-gdb` instead of just `gdb`?**
    *   **A:** While standard `gdb` can debug Rust binaries, it displays Rust standard types (Vec, String, HashMap) as their raw memory layout. `rust-gdb` loads pretty-printers that format these types into readable output, saving developers time decoding memory manually.

2.  **Q: How do you debug a production binary where `cargo run --release` crashes?**
    *   **A:** By default, release builds strip debug symbols to reduce binary size. To debug effectively, you must enable debug symbols in the release profile by adding `debug = true` to the `[profile.release]` section of `Cargo.toml`, then rebuild and use `rust-gdb`.

3.  **Q: Can `rust-gdb` debug asynchronous code?**
    *   **A:** Yes, but it can be challenging. Standard GDB doesn't understand the concept of "Tasks" in async runtimes like Tokio. It sees threads. You often need to inspect the state of the Future state machines manually or use specialized tools (like `tokio-console`) alongside GDB.
