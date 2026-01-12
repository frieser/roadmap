# rust-lldb
---
---

## Summary
`rust-lldb` is the LLDB equivalent of `rust-gdb`. It is a wrapper script that launches the LLDB debugger with Rust-specific formatting configurations. It is the preferred and native debugger for macOS users (where GDB is often not supported or requires code signing) and serves as the backend for many VS Code debugging extensions.

## Detailed Explanation

### Platform Preference
-   **macOS**: **Primary Choice.** Apple's development environment (Xcode) is built on LLVM/LLDB. GDB is difficult to run due to system integrity protections.
-   **Linux**: **Alternative Choice.** While GDB is standard, LLDB is fully supported and often preferred by users of Clang or VS Code (CodeLLDB).
-   **Windows**: Generally uses the MSVC debugger, but LLDB is available via MinGW or inside WSL.

### Basic Usage
1.  **Compile**:
    ```bash
    cargo build
    ```
2.  **Start Debugger**:
    ```bash
    rust-lldb target/debug/my_project
    ```
3.  **Command Translation**:
    *   Instead of `print` (which evaluates expressions), modern LLDB usage often prefers `frame variable` (or `v`) for inspecting variables simply and quickly.

### Key LLDB Commands vs GDB
| Action | GDB Command | LLDB Command |
| :--- | :--- | :--- |
| **Set Breakpoint** | `b main.rs:10` | `breakpoint set --file main.rs --line 10` (or `br s -f main.rs -l 10`) |
| **Run** | `run` | `process launch` (or `r`) |
| **Next Line** | `next` | `thread step-over` (or `n`) |
| **Step In** | `step` | `thread step-in` (or `s`) |
| **Print Var** | `print x` | `frame variable x` (or `v x`) |
| **Evaluate Expr** | `p x + 1` | `expression -- x + 1` (or `p x + 1`) |
| **Backtrace** | `bt` | `thread backtrace` (or `bt`) |
| **List Frames** | `info threads` | `thread list` |

## Interview Questions

1.  **Q: What is the main advantage of using `v` (frame variable) over `p` (print) in LLDB?**
    *   **A:** `v` accesses the variable's memory directly using debug symbols, which is fast and side-effect free. `p` compiles and executes a small expression to evaluate the result. `v` handles Rust variables more reliably without potentially altering the program state.

2.  **Q: How does `rust-lldb` relate to VS Code debugging?**
    *   **A:** The popular "CodeLLDB" extension for VS Code uses LLDB as its underlying engine. It essentially performs the same task as `rust-lldb` (loading pretty-printers) but wraps the command-line interface in the IDE's GUI adapter.

3.  **Q: Can you set conditional breakpoints in `rust-lldb`?**
    *   **A:** Yes. You can use `breakpoint modify -c "condition" <breakpoint-id>`. For example: `br s -f main.rs -l 20` followed by `br mod -c "i == 5" 1`. This stops execution only when the variable `i` equals 5.
