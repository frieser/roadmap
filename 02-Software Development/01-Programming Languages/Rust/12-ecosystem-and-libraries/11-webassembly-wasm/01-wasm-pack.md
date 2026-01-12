# Wasm-Pack (Sunset)
---
---

## Summary
**Wasm-Pack** was historically the standard tool for building, testing, and publishing Rust-generated WebAssembly. As of 2025/2026, it is largely considered **maintenance-only** or **sunsetted** in favor of more modern tools like **Trunk** or integrated bundler plugins (Vite/Webpack).

## Detailed Explanation

### Historical Role
Wasm-Pack was crucial in the early days because it handled the complex steps of compiling Rust to WASM, generating the JavaScript glue code (via `wasm-bindgen`), and packaging it for npm.

### Modern Alternatives
*   **Trunk**: A zero-config WASM web application bundler. It's the standard for building pure Rust web apps (Leptos, Yew).
*   **Vite Plugins**: `vite-plugin-wasm` allows seamless integration of Rust WASM into modern JS frontends.

### Use Cases
*   **Legacy Projects**: Maintaining older libraries published to npm.

## Interview Questions

1.  **Q: If I'm starting a new Rust Web App (like Leptos) today, should I use `wasm-pack`?**
    *   **A:** Generally, no. Frameworks like Leptos or Yew recommend using **Trunk** (`cargo-leptos` uses it internally). Trunk handles the build process, serving, and hot-reloading much more smoothly for full applications.

2.  **Q: What does `wasm-pack build --target web` do?**
    *   **A:** It compiles the Rust code to `.wasm` and generates a JavaScript wrapper file that natively imports the WASM module. This target is for direct use in a browser (without a bundler like Webpack).
