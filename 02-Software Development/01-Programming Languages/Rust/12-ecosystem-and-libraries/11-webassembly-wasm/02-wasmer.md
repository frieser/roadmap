# Wasmer
---
---

## Summary
**Wasmer** is a standalone WebAssembly runtime that enables running WASM binaries outside the browser—on servers, edge devices, or as a plugin system for other apps. It supports WASI (WebAssembly System Interface), allowing WASM modules to access files and networks safely.

## Detailed Explanation

### Core Philosophy
"Run any code, anywhere." Wasmer allows you to take code compiled to WASM (from Rust, C++, Go, Python) and run it on any OS (Linux, Windows, macOS) with near-native speed.

### Key Features
*   **Universal Runtime**: Run WASM on the server.
*   **WASI Support**: Provides system calls (I/O, Time) to WASM modules.
*   **Plug-in System**: Use Wasmer to embed a safe scripting engine into your Rust application.
*   **Package Manager**: `wapm` (now merged into Wasmer ecosystem) for distributing WASM binaries.

### Use Cases
*   **Edge Computing**: Deploying lightweight functions to the edge.
*   **Safe Plugins**: Allowing users to write plugins for your app in any language, running them in a sandboxed WASM environment.

### Code Example (Embedding Wasmer)
*Dependencies: `wasmer`*

```rust
use wasmer::{Store, Module, Instance, Value, imports};

fn main() -> anyhow::Result<()> {
    // 1. Create a Store
    let mut store = Store::default();

    // 2. Compile WASM bytes
    let wasm_bytes = r#"
        (module
          (func $sum (export "sum") (param i32 i32) (result i32)
            local.get 0
            local.get 1
            i32.add))
    "#;
    let module = Module::new(&store, wasm_bytes)?;

    // 3. Create an Import Object (empty here)
    let import_object = imports! {};

    // 4. Instantiate the module
    let instance = Instance::new(&mut store, &module, &import_object)?;

    // 5. Call the exported function
    let sum = instance.exports.get_function("sum")?;
    let results = sum.call(&mut store, &[Value::I32(5), Value::I32(37)])?;

    println!("Results: {:?}", results); // [I32(42)]
    Ok(())
}
```

## Interview Questions

1.  **Q: What is WASI?**
    *   **A:** WASI (WebAssembly System Interface) is a standard API that allows WebAssembly modules to access operating system features like files, networking, and system clocks in a portable and secure way. Without WASI, standard WASM cannot interact with the outside world.

2.  **Q: How does Wasmer ensure security?**
    *   **A:** WebAssembly is sandboxed by design. A module cannot access memory outside its own linear memory block. Wasmer enforces this sandbox and only allows access to system resources (via WASI) if explicitly granted by the host application.
