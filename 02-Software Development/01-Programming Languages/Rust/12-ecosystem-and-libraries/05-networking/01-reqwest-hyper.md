# Reqwest & Hyper
---
---

## Summary
**Reqwest** is the most popular high-level HTTP client for Rust. It provides a convenient, "batteries-included" interface for making requests. **Hyper** is the low-level HTTP implementation that powers Reqwest (and Axum). While you usually use Reqwest directly, Hyper is the engine under the hood.

## Detailed Explanation

### Reqwest
*   **Philosophy**: "Easy things should be easy." It abstracts away the complexities of connection pooling, redirect handling, and TLS setup.
*   **Features**:
    *   Async and Blocking clients.
    *   JSON support via Serde.
    *   Multipart support.
    *   WASM support (works in the browser!).

### Hyper
*   **Philosophy**: "Correct and fast HTTP implementation." It is a low-level library intended for building libraries or high-performance servers, not for direct end-user application usage.

### Use Cases
*   **Reqwest**: Consuming external APIs, web scraping, microservice communication.
*   **Hyper**: Building a new web framework, or a specialized proxy server.

### Code Example (Reqwest)
*Dependencies: `reqwest`, `tokio`, `serde`, `serde_json`*

```rust
use serde::Deserialize;

#[derive(Deserialize, Debug)]
struct HttpBinResponse {
    url: String,
    origin: String,
}

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    // Making a simple GET request
    let resp: HttpBinResponse = reqwest::Client::new()
        .get("https://httpbin.org/get")
        .header("User-Agent", "MyRustApp")
        .send()
        .await?
        .json()
        .await?;

    println!("Response from {}", resp.url);
    Ok(())
}
```

## Interview Questions

1.  **Q: Why should you reuse the `reqwest::Client` instead of creating a new one for each request?**
    *   **A:** `reqwest::Client` holds an internal connection pool. Reusing it allows the application to keep TCP connections open (Keep-Alive) and reuse them for subsequent requests, significantly reducing latency and system resource usage (handshakes are expensive).

2.  **Q: How do you enable the blocking API in Reqwest?**
    *   **A:** You must enable the `blocking` feature in your `Cargo.toml` (`reqwest = { version = "...", features = ["blocking"] }`). This provides a synchronous `reqwest::blocking::Client` for use cases where an async runtime is not available or desired.

3.  **Q: What is the relationship between Reqwest and Hyper?**
    *   **A:** Reqwest is a high-level wrapper around Hyper. Hyper handles the raw HTTP protocol (parsing bytes, managing connections), while Reqwest adds user-friendly features like cookies, redirects, automatic JSON serialization, and TLS configuration.
