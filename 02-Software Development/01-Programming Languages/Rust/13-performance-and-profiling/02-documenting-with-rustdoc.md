# Documenting with Rustdoc
---
---

## Summary
**Rustdoc** is the built-in documentation tool for Rust. It extracts documentation comments from your source code and generates high-quality HTML documentation. A unique and powerful feature of Rustdoc is **Documentation Tests (Doc Tests)**, where code examples in your documentation are compiled and run as tests, ensuring your documentation never goes out of date.

## Detailed Explanation

### Key Features
- **Markdown Support**: Documentation comments support full CommonMark markdown (bold, lists, code blocks).
- **Intra-doc Links**: You can link to other structs, traits, or functions using standard Rust paths, e.g., `[MyStruct]`.
- **Doc Tests**: Code blocks inside documentation are treated as executable tests.
- **Standardized Output**: Generates a consistent, searchable HTML interface for all crates (seen on docs.rs).

### Documentation Comments
Rust uses two types of documentation comments:
1.  **Outer Doc Comments (`///`)**: Document the item *following* the comment (e.g., a function or struct).
2.  **Inner Doc Comments (`//!`)**: Document the item *containing* the comment (typically used at the top of `lib.rs` or `mod.rs` to document the crate or module itself).

### How to Use
- **Generate Docs**: `cargo doc`
- **Open Docs**: `cargo doc --open` (builds and opens in browser)
- **Run Doc Tests**: `cargo test` (runs both unit tests and doc tests)

### Code Example: Writing Effective Docs
```rust
/// A representation of a specialized container.
///
/// # Examples
///
/// ```
/// use my_crate::Container;
///
/// let c = Container::new(5);
/// assert_eq!(c.value(), 5);
/// ```
pub struct Container {
    val: i32,
}

impl Container {
    /// Creates a new [`Container`] with the given value.
    ///
    /// # Arguments
    ///
    /// * `val` - The integer value to store.
    pub fn new(val: i32) -> Self {
        Container { val }
    }

    /// Returns the contained value.
    pub fn value(&self) -> i32 {
        self.val
    }
}
```

### Best Practices
1.  **Use Sections**: Use standard headers like `# Examples`, `# Panics`, `# Errors`, and `# Safety` (for unsafe functions).
2.  **Link Everything**: Use intra-doc links (`[`Option`]`) instead of writing "Option".
3.  **Hide Boilerplate**: In examples, use `#` to hide lines from the display but keep them for the test runner.
    ```rust
    /// ```
    /// # fn main() -> Result<(), Box<dyn std::error::Error>> {
    /// let mut f = File::open("foo.txt")?;
    /// # Ok(())
    /// # }
    /// ```
    ```

## Interview Questions

**Q: What is the difference between `///` and `//!` comments?**
**A:** `///` (triple slash) documents the item that follows it, such as a function definition or struct declaration. `//!` (double slash bang) documents the item that contains it, usually the crate root (`lib.rs`) or a module file (`mod.rs`), providing high-level documentation for that module.

**Q: Explain how Doc Tests ensure documentation quality.**
**A:** When you run `cargo test`, Rustdoc extracts all code blocks from your documentation comments and compiles them as executable tests. If your example code doesn't compile or panics, the test suite fails. This guarantees that the examples in your documentation are always valid and up-to-date with your API.

**Q: How do you link to another struct or function within the documentation?**
**A:** You use **Intra-doc links**. Instead of using a standard Markdown link URL, you put the Rust path in brackets, like `[MyStruct]` or `[std::io::Error]`. Rustdoc resolves these paths at compile time and generates the correct HTML links.

**Q: How can you hide setup code in a documentation example while still running it?**
**A:** You can prefix lines with a hash symbol (`#`) inside the code block. These lines will be compiled and run by the test runner but will not be visible in the generated HTML documentation. This is useful for hiding boilerplate imports or main function wrappers.
