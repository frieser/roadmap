# RustCrypto
---
---

## Summary
**RustCrypto** is an organization that maintains a collection of pure-Rust cryptographic crates. It provides a set of standard traits (like `Digest`, `Aead`, `Cipher`) that allow different implementations to be swapped easily.

## Detailed Explanation

### Core Philosophy
"Pure Rust, Modular, and Trait-based." The goal is to provide implementations that are written entirely in safe Rust (where possible), are `no_std` compatible (for embedded use), and share a common interface.

### Key Features
*   **Modularity**: You only pull in the crate you need (e.g., `sha2`, `aes`, `hmac`).
*   **Traits**: The `digest` and `aead` crates define standard interfaces.
*   **Embedded Friendly**: Most crates work without the standard library (`no_std`).

### Code Example
*Dependencies: `sha2`, `digest`*

```rust
use sha2::{Sha256, Digest};

fn main() {
    // create a Sha256 object
    let mut hasher = Sha256::new();

    // write input message
    hasher.update(b"hello world");

    // read hash digest and consume hasher
    let result = hasher.finalize();

    println!("Hash: {:x}", result);
}
```

## Interview Questions

1.  **Q: Why is `no_std` support important for crypto libraries?**
    *   **A:** It allows the cryptography to be used in embedded devices (microcontrollers), WASM modules, and operating system kernels where the full standard library (and heap allocation) might not be available.

2.  **Q: What is the `Digest` trait?**
    *   **A:** It is a trait from the `digest` crate that abstracts over cryptographic hash functions. Any function accepting `impl Digest` can accept `Sha256`, `Sha512`, `Blake2`, etc., making code reusable.
