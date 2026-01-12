# Wasm-Bindgen
---
---

## Summary
**wasm-bindgen** is the core library that bridges the gap between Rust and JavaScript. It facilitates high-level interactions between the two languages, allowing Rust to call JS APIs (like the DOM) and JS to call Rust functions with complex types (Strings, Objects, Classes) rather than just numbers.

## Detailed Explanation

### Core Philosophy
WASM natively only understands numbers (i32, f64). `wasm-bindgen` generates the "glue code" that serializes complex objects into memory and deserializes them on the other side. It makes Rust feel like a first-class citizen in the JS ecosystem.

### Key Features
*   **`#[wasm_bindgen]` Macro**: Expose Rust functions/structs to JS.
*   **JS Sys / Web Sys**: Crates that provide bindings to standard JS objects (`Array`, `Date`) and Web APIs (`document`, `window`, `fetch`).
*   **Type Conversion**: Automatically handles string/array passing.

### Use Cases
*   **Frontend Frameworks**: Leptos and Yew rely heavily on this to manipulate the DOM.
*   **Hybrid Libraries**: Writing a CPU-intensive algorithm (like image processing) in Rust and calling it from a React app.

### Code Example
*Dependencies: `wasm-bindgen`*

```rust
use wasm_bindgen::prelude::*;

// Import the `window.alert` function from JS
#[wasm_bindgen]
extern "C" {
    fn alert(s: &str);
}

// Export a Rust function to JS
#[wasm_bindgen]
pub fn greet(name: &str) {
    alert(&format!("Hello, {}!", name));
}
```

## Interview Questions

1.  **Q: How are Strings passed between Rust and JavaScript in `wasm-bindgen`?**
    *   **A:** Since WASM memory is a flat array of bytes separate from the JS heap, passing a string involves: 1) Allocating memory in WASM, 2) Copying the string bytes into that memory, 3) Passing the pointer and length to JS, 4) JS using a TextDecoder to read those bytes. `wasm-bindgen` handles this dance automatically.

2.  **Q: What is `web-sys`?**
    *   **A:** `web-sys` is a crate that provides raw bindings to all the Web APIs (DOM, CSS, WebGL, Audio, etc.) generated from the official WebIDL specifications. It allows you to write `window.document().get_element_by_id(...)` in Rust.

3.  **Q: Can I pass a Rust struct to JavaScript?**
    *   **A:** Yes, if you annotate the struct with `#[wasm_bindgen]`. The struct will be wrapped in a JavaScript class. However, the data lives in Rust memory; the JS object is just a handle (pointer) to it. You must be careful with lifetimes and explicit deallocation (calling `.free()`) in some contexts, though `wasm-bindgen` handles much of this.
