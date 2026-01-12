# Tauri
---
---

## Summary
Tauri is a toolkit for building tiny, fast binaries for all major desktop platforms. It allows you to build the frontend of your application using web technologies (HTML, JS, CSS, or frameworks like React/Svelte) and the backend in Rust. It distinguishes itself from Electron by using the OS's native webview (WebView2 on Windows, WebKit on macOS/Linux), resulting in significantly smaller binaries and lower RAM usage.

## Detailed Explanation

### Core Philosophy
"Smaller, Faster, More Secure." Tauri aims to provide the developer experience of Electron without the resource bloat. By relying on the system's existing webview, it doesn't need to bundle Chromium. It also enforces a strict security model where the frontend cannot arbitrarily access system APIs; it must invoke Rust functions to do so.

### Key Features
*   **Tiny Binaries**: Hello World is < 600KB (vs Electron's ~100MB).
*   **Security Isolation**: The frontend runs in a sandboxed webview. Access to file system or shell is strictly controlled via an allowlist.
*   **Polyglot**: Use any frontend stack you want.
*   **Mobile Support**: Tauri v2 adds support for iOS and Android.

### Use Cases
*   **Cross-Platform Desktop Apps**: Tools like cryptographic wallets, system optimizers, or chat apps.
*   **Hybrid Apps**: Apps that share UI code with a web version.

### Code Example
*Dependencies: `tauri`*

```rust
// main.rs - The Rust Backend
#![cfg_attr(
    all(not(debug_assertions), target_os = "windows"),
    windows_subsystem = "windows"
)]

// A command callable from the frontend
#[tauri::command]
fn greet(name: &str) -> String {
    format!("Hello, {}! You've been greeted from Rust!", name)
}

fn main() {
    tauri::Builder::default()
        // Register the command
        .invoke_handler(tauri::generate_handler![greet])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

```javascript
// frontend.js - calling the Rust function
import { invoke } from '@tauri-apps/api/tauri';

invoke('greet', { name: 'World' })
  .then((response) => console.log(response));
```

## Interview Questions

1.  **Q: How does Tauri achieve such small binary sizes compared to Electron?**
    *   **A:** Electron bundles the entire Chromium browser engine and Node.js runtime with every application. Tauri reuses the Webview already installed on the user's operating system (Edge WebView2 on Windows, WebKit on macOS/Linux). This removes the need to ship a browser engine.

2.  **Q: What is the "Isolation Pattern" in Tauri?**
    *   **A:** It is a security feature where the IPC (Inter-Process Communication) bridge between the Webview and Rust is injected into a separate, isolated context. This prevents untrusted scripts loaded in the webview from accessing sensitive API tokens or the IPC mechanism directly.

3.  **Q: Can you use Node.js modules in Tauri?**
    *   **A:** Not directly in the final binary. The backend is Rust, not Node.js. However, during development, you can use Node.js for bundling your frontend assets (Vite, Webpack). If you need OS-level functionality at runtime, you write it in Rust.
