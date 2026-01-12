# Ring
---
---

## Summary
**Ring** is a high-performance cryptography library for Rust. It focuses on providing a safe, easy-to-use API for a specific set of standard cryptographic algorithms (AES-GCM, SHA-2, ECDSA, Ed25519). It is widely used in the ecosystem (including by `rustls`).

## Detailed Explanation

### Core Philosophy
Ring prioritizes **correctness** and **speed**. It exposes a limited set of "good" algorithms to prevent users from choosing insecure options (like MD5 or RC4). Internally, it uses a mix of Rust and assembly (derived from BoringSSL) to ensure constant-time execution and maximum performance.

### Key Features
*   **Safe API**: Hard to misuse.
*   **Performance**: Highly optimized assembly backends.
*   **Standard**: Powers much of the Rust secure networking stack.

### Use Cases
*   **TLS**: Underlying crypto for HTTPS.
*   **Authentication**: Password hashing (PBKDF2), Token signing (JWT).

### Code Example
*Dependencies: `ring`*

```rust
use ring::digest::{Context, SHA256};
use ring::rand::SystemRandom;

fn main() {
    // 1. Hashing
    let mut context = Context::new(&SHA256);
    context.update(b"hello world");
    let digest = context.finish();
    println!("SHA-256: {:?}", digest.as_ref());

    // 2. Random bytes
    let rng = SystemRandom::new();
    // ... usage of rng to generate keys
}
```

## Interview Questions

1.  **Q: Why does Ring use C/Assembly code?**
    *   **A:** Cryptography requires constant-time execution to prevent side-channel attacks (timing attacks). It is historically difficult to guarantee constant-time behavior in high-level languages due to compiler optimizations. Ring reuses the battle-tested assembly code from BoringSSL (Google's OpenSSL fork) to ensure both security and speed.

2.  **Q: How does Ring differ from the `rust-crypto` traits?**
    *   **A:** Ring is a monolithic, opinionated library. `rust-crypto` is a collection of traits (interfaces) and pure-Rust implementations. Ring is often preferred for production apps needing speed/security auditing, while RustCrypto is great for `no_std` environments or when you need an algorithm Ring doesn't support.
