#Rust
---
---

## Summary
The standard way to install Rust is via `rustup`, a toolchain multiplexer. It installs `rustc` (the compiler), `cargo` (the package manager), and other standard tools. This approach ensures you can easily switch between stable, beta, and nightly channels and keep your environment up to date.

## Detailed Explanation

### The Rust Toolchain
When you install Rust, you get three main components:
1. **rustup**: The installer and version manager. It manages different versions of Rust (stable, beta, nightly) and targets (like `wasm32-unknown-unknown`).
2. **rustc**: The actual compiler. You rarely call this directly; `cargo` calls it for you.
3. **cargo**: The build system and package manager. You will use this for almost everything: creating projects, building code, running tests, and managing dependencies.

### Installation (Linux/macOS)
The official one-liner to install Rust:
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
This script adds `cargo`, `rustc`, and `rustup` to your `PATH` (usually in `~/.cargo/bin`).

### Common Commands

| Command | Description |
|---------|-------------|
| `rustup update` | Updates Rust to the latest version. |
| `rustc --version` | Checks the compiler version. |
| `cargo new <name>` | Creates a new binary project. |
| `cargo build` | Compiles the project (Debug mode). |
| `cargo run` | Compiles and runs the project immediately. |
| `cargo check` | Checks for errors without generating a binary (much faster). |

### Hello World
After installation, verify it works:

```bash
cargo new hello_rust
cd hello_rust
cargo run
# Output: Hello, world!
```

## Interview Questions

1. **What is the difference between `rustc` and `cargo`?**
   - `rustc` is the compiler that transforms Rust code into binary. `cargo` is the package manager and build tool that orchestrates `rustc`, manages dependencies, and runs tests.

2. **Why is `cargo check` useful?**
   - `cargo check` analyzes the code for errors but skips the final code generation step. It is significantly faster than `cargo build`, making it ideal for rapid feedback during development.

3. **How do you update Rust?**
   - By running `rustup update`. Since Rust releases a new stable version every 6 weeks, this is a frequently used command.
