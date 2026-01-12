#Rust
---
---

## Summary
Procedural Macros ("proc-macros") are advanced metaprogramming tools that act as compiler plugins. Unlike declarative macros which use pattern matching, procedural macros accept a stream of tokens (`TokenStream`) as input and produce a new stream of tokens as output. They are typically used for generating code based on attributes or struct definitions.

## Detailed Explanation

### 1. Three Types of Proc-Macros
1.  **Custom Derive** (`#[derive(MyTrait)]`): Used to automatically implement traits for structs and enums. Input is the struct/enum definition.
2.  **Attribute-like** (`#[route("/")]`): Custom attributes attached to items (functions, structs, modules). Can modify or replace the item.
3.  **Function-like** (`sql!("SELECT...")`): Look like declarative macros but are more powerful. Can take any arbitrary token stream as input.

### 2. The Ecosystem
Writing raw proc-macros is hard. The community relies on two essential crates:
- **`syn`**: A parser that converts the raw `TokenStream` into a manageable Abstract Syntax Tree (AST) (e.g., `DeriveInput`).
- **`quote`**: A quasi-quoting library that allows you to write Rust code templates (using the `#var` syntax for interpolation) and convert them back into a `TokenStream`.

### 3. Crate Restrictions
Procedural macros **must** be defined in a separate crate with `proc-macro = true` in `Cargo.toml`. This is because the compiler needs to compile the macro code before compiling the crate that uses it.

## Rust Application

### Conceptual Example: A `Hello` Derive Macro
Imagine we want to auto-generate a `hello()` method for any struct.

**1. The Macro Definition (in `hello_macro` crate):**
```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(Hello)]
pub fn hello_derive(input: TokenStream) -> TokenStream {
    // Parse the input tokens into a syntax tree
    let ast = parse_macro_input!(input as DeriveInput);

    // Get the name of the struct (e.g., "User")
    let name = &ast.ident;

    // Generate the implementation
    let gen = quote! {
        impl Hello for #name {
            fn hello() {
                println!("Hello, I am {}!", stringify!(#name));
            }
        }
    };

    // Return the generated tokens
    gen.into()
}
```

**2. Usage (in main crate):**
```rust
use hello_macro::Hello;

trait Hello {
    fn hello();
}

#[derive(Hello)]
struct User;

fn main() {
    User::hello(); // Output: "Hello, I am User!"
}
```

## Interview Questions

### Q: Why must procedural macros be in a separate crate?
**A:** Procedural macros are essentially compiler plugins. The Rust compiler must build and execute the macro code *during* the compilation of the main project. Therefore, the macro crate must be compiled independently and linked into the compiler process before the main crate can be processed.

### Q: What is the difference between `syn` and `quote`?
**A:** `syn` is a **parser**: it takes a stream of tokens and turns it into a structured syntax tree (AST) that you can inspect (e.g., "get all fields of this struct"). `quote` is a **generator**: it takes Rust-like syntax templates and variables and turns them back into a stream of tokens for the compiler.

### Q: Are procedural macros hygienic?
**A:** No, procedural macros are generally **unhygienic**. They act on raw tokens. If your macro generates code that uses `String`, and the user has shadowed `String` in their scope, the macro will break. Best practice is to use fully qualified paths (e.g., `::std::string::String`) in generated code to avoid collisions.
