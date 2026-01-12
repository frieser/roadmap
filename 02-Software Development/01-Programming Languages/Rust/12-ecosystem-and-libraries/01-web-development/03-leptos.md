# Leptos
---
---

## Summary
Leptos is a modern, high-performance full-stack web framework for Rust that brings fine-grained reactivity to the web. Unlike React-like frameworks that use a Virtual DOM, Leptos uses Signals to surgically update the DOM, resulting in exceptional performance. It supports Server-Side Rendering (SSR) with hydration out of the box, making it a competitor to frameworks like Next.js or SolidJS.

## Detailed Explanation

### Core Philosophy
Leptos is built on the belief that "web apps should be fast by default." It adopts the architecture of **SolidJS**—using signals to track dependencies and updating only the exact text node or attribute that changed. This removes the overhead of diffing a Virtual DOM tree.

### Key Features
*   **Fine-Grained Reactivity**: Uses `Signal`, `Memo`, and `Effect` to update the DOM directly.
*   **Isomorphic / Full-Stack**: Write your backend (in Axum or Actix) and frontend (WASM) in a single Rust codebase.
*   **Server Functions**: Call server-side Rust functions directly from client-side code as if they were async local functions (`#[server]`).
*   **Hydration**: Sends HTML from the server for instant First Contentful Paint, then "wakes up" with WASM.
*   **Typed View Macro**: The `view!` macro offers a JSX-like experience but with Rust's type safety.

### Use Cases
*   **Performance-Critical Dashboards**: Where re-rendering the whole tree is too expensive.
*   **Single Page Applications (SPAs)**: Complex interactive apps.
*   **Full-Stack Rust Projects**: Teams that want to share types and validation logic between front and back ends seamlessly.

### Code Example
*Dependencies: `leptos`*

```rust
use leptos::*;

#[component]
pub fn SimpleCounter(initial_value: i32) -> impl IntoView {
    // Create a reactive signal with read and write primitives
    let (count, set_count) = create_signal(initial_value);

    view! {
        <div class="counter-container">
            <button
                // Event listener
                on:click=move |_| set_count.update(|n| *n -= 1)
            >
                "-1"
            </button>
            
            // The text node here updates automatically when `count` changes
            <span>"Value: " {move || count.get()}</span>
            
            <button
                on:click=move |_| set_count.update(|n| *n += 1)
            >
                "+1"
            </button>
        </div>
    }
}
```

## Interview Questions

1.  **Q: How does Leptos differ from Yew (another Rust frontend framework)?**
    *   **A:** Yew uses a Virtual DOM (VDOM) similar to React, where it diffs trees to decide what to update. Leptos uses fine-grained reactivity (Signals) similar to SolidJS, where components run once to set up the dependency graph, and updates touch the DOM directly without diffing. This generally makes Leptos faster and more memory-efficient.

2.  **Q: What are "Server Functions" in Leptos?**
    *   **A:** Server Functions are Rust functions annotated with `#[server]`. They run exclusively on the server but can be called from the client code. The compiler generates the API endpoints and network glue code (serialization/deserialization) automatically, making the network boundary transparent to the developer.

3.  **Q: Explain the concept of "Hydration" in Leptos.**
    *   **A:** Hydration is the process where the server renders the initial HTML state of the app (SSR) and sends it to the browser. The browser displays this immediately. Then, the WebAssembly bundle loads and attaches event listeners to the existing HTML elements, making the page interactive without throwing away the existing DOM.

4.  **Q: Why do we need `move` closures in Leptos event listeners?**
    *   **A:** In Rust, closures capture their environment. Since the `view!` macro creates code that might outlive the current function scope (because it's attached to the DOM), we often need to `move` ownership of the signals (which are `Copy`-able IDs) into the closure so they are valid when the event fires later.
