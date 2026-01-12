#Rust
---
---

## Summary
Error propagation allows a function to pass an error up to its caller instead of handling it immediately. The `?` operator is a shorthand for "return the error if this fails, otherwise give me the value," making error handling concise and readable.

## Detailed Explanation

### The `?` Operator
Placed after a `Result` (or `Option`) expression.
- If the value is `Ok(x)`, the expression evaluates to `x`.
- If the value is `Err(e)`, the function returns `Err(e)` immediately.

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();
    // If open fails, return error. If works, assign to f.
    let mut f = File::open("hello.txt")?; 
    
    // If read fails, return error. If works, return Ok(s).
    f.read_to_string(&mut s)?;
    
    Ok(s)
}
```

### Main returning Result
`main` can return `Result<(), E>`. This allows using `?` in the main function. If main returns `Err`, the program exits with a non-zero status code and prints the error debug info.

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let f = File::open("hello.txt")?;
    Ok(())
}
```

### Custom Error Types
For libraries, defining custom error types is common to provide meaningful context.
- The `thiserror` crate is widely used to derive the `Error` trait easily.
- The `anyhow` crate is popular for applications where you just want to handle "any error".

```rust
#[derive(Debug)]
enum MyError {
    Io(std::io::Error),
    Parse(std::num::ParseIntError),
}
```

## Interview Questions

1. **What does the `?` operator do?**
   - It propagates errors. It unwraps the `Ok` value if successful; if it encounters an `Err`, it returns that error from the *current function* immediately, converting it (via `From` trait) to the function's return error type if necessary.

2. **Can you use `?` in a function that returns `()`?**
   - No. The function must return a `Result` (or `Option`) compatible with the type being operated on. If you try to use `?` in a void function, the compiler will error.

3. **What is the role of the `From` trait in error propagation?**
   - When using `?`, if the error type returned by the called function differs from the calling function's return type, Rust tries to use `From::from` to convert the error. This allows wrapping lower-level errors (like `io::Error`) into higher-level custom errors automatically.
