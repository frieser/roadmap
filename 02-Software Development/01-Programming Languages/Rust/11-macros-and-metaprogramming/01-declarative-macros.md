#Rust
---
---

## Summary
Declarative macros (also known as "macros by example") are the most common form of metaprogramming in Rust. Defined using `macro_rules!`, they allow you to write code that writes other code by matching patterns in the input syntax and expanding them into valid Rust code at compile time. They are powerful tools for reducing boilerplate and creating simple Domain Specific Languages (DSLs).

## Detailed Explanation

### 1. Structure of `macro_rules!`
A declarative macro works like a `match` expression. It has multiple "arms", each with a pattern to match and a block of code to generate.
- **Matchers**: Variables starting with `$` that capture parts of the code.
  - `$ident`: An identifier (variable/function name).
  - `$expr`: An expression (e.g., `2 + 2`).
  - `$ty`: A type (e.g., `i32`).
  - `$tt`: A token tree (a catch-all for tokens).
- **Repetition**: Handling variable numbers of arguments.
  - `$(...)*`: Match zero or more times.
  - `$(...)+`: Match one or more times.
  - `$(...),*`: Match zero or more times, separated by commas.

### 2. Hygiene
Rust macros are "partially hygienic".
- **Hygienic**: Local variables defined inside a macro won't clash with variables in the scope where the macro is called.
- **Unhygienic**: Items (functions, structs) defined by the macro *are* visible to the outer scope.

### 3. Scoping and Exporting
- `#[macro_export]`: Makes the macro available to other crates.
- `$crate`: A special variable used to refer to the current crate's root, ensuring paths (like `::std::vec::Vec`) resolve correctly regardless of where the macro is used.

## Rust Application

### Example 1: A `vec!`-like Macro
A macro that creates a vector and pushes elements into it.

```rust
#[macro_export]
macro_rules! my_vec {
    // Case 1: Empty vector
    () => {
        std::vec::Vec::new()
    };
    // Case 2: Comma-separated elements
    // Match an expression ($x:expr), repeated zero or more times (*), separated by commas (,)
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = std::vec::Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}

fn main() {
    let v = my_vec![1, 2, 3];
    // Expands to:
    // {
    //     let mut temp_vec = std::vec::Vec::new();
    //     temp_vec.push(1);
    //     temp_vec.push(2);
    //     temp_vec.push(3);
    //     temp_vec
    // }
}
```

### Example 2: Simple Logic DSL
A macro that defines a custom syntax for operations.

```rust
macro_rules! calculate {
    (add $a:expr, $b:expr) => {
        $a + $b
    };
    (square $a:expr) => {
        $a * $a
    };
}

fn main() {
    let sum = calculate!(add 5, 10); // 15
    let sq = calculate!(square 4);   // 16
}
```

## Interview Questions

### Q: What is the difference between a declarative macro and a function?
**A:** A function operates on values at runtime and has a fixed signature (types and number of arguments). A macro operates on syntax (tokens) at compile time, can accept a variable number of arguments, and can generate new code (like struct definitions) that functions cannot.

### Q: What is macro hygiene?
**A:** Hygiene prevents variable name collisions. If a macro defines `let x = 5;`, it won't overwrite a variable named `x` in the user's code calling the macro. However, Rust macros are only *partially* hygienic; items like helper functions defined in the macro can pollute the global namespace.

### Q: What is a "Fragment Specifier" or "Matcher"?
**A:** It tells the macro parser what kind of syntax to expect. Common ones are `$expr` (expression), `$ident` (identifier), `$ty` (type), and `$stmt` (statement). Using the wrong matcher (e.g., trying to match a struct definition with `$expr`) will cause a compile error.
