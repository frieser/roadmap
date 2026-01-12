#Rust
---
---

## Summary
Rust is a compiled language, so it doesn't have a built-in REPL like Python or Ruby. However, the official **Rust Playground** serves as an instant online environment, and third-party tools like **evcxr** provide a local REPL experience for quick experimentation.

## Detailed Explanation

### 1. The Rust Playground
[play.rust-lang.org](https://play.rust-lang.org/) is an essential tool for the community.
- **Sharing**: You can write code and generate a permalink to share snippets (common on StackOverflow and Discord).
- **Tools**:
  - **Expand Macros**: See what `println!` or `#[derive]` actually generate.
  - **ASM/LLVM IR**: View the generated Assembly or LLVM Intermediate Representation.
  - **MIRI**: Run code under Miri to detect undefined behavior (UB).
- **Limitations**: No file I/O, no networking, limited external crates (only top 100 crates available).

### 2. Local REPL: evcxr
Since `cargo run` takes time to compile, a Read-Eval-Print Loop (REPL) is useful for learning syntax or testing small logic.
- **Tool**: `evcxr_repl` (maintained by Google).
- **Installation**:
  ```bash
  cargo install evcxr_repl
  ```
- **Usage**:
  ```bash
  $ evcxr
  >> let x = 5;
  >> println!("{}", x);
  5
  ```
- **Jupyter Notebooks**: There is also a Jupyter kernel (`evcxr_jupyter`) allowing Rust to be used in data science notebooks.

### 3. Alternative: Cargo Scripts
For single-file experiments that are too big for a REPL but too small for `cargo new`:
- You can treat a `.rs` file as a script using `rust-script` (unofficial but popular).
- *Future*: RFC 3502 (embedded manifest) is bringing single-file packages directly to Cargo.

## Interview Questions

1. **Does Rust have a built-in REPL?**
   - No, because it is a compiled language. However, tools like `evcxr` provide REPL functionality, and the online Rust Playground is widely used for snippets.

2. **What is the Rust Playground useful for besides running code?**
   - It is useful for inspecting macro expansion (to understand how macros work), viewing generated assembly (for optimization checks), and checking for Undefined Behavior using Miri.

3. **How can you quickly test a small snippet of Rust code locally without creating a full project?**
   - You can use `evcxr_repl` for interactive testing, or simply create a file and run it with `rustc main.rs` (though using `cargo` is generally preferred even for small things to handle dependencies).
