# Axum
---
---

## Summary
Axum is an ergonomic and modular web application framework for Rust, built and maintained by the Tokio team. It focuses on providing a user-friendly API without relying heavily on macros, leveraging Rust's type system and traits to handle routing and request extraction. It is fully compatible with the `tower` ecosystem, allowing developers to reuse robust middleware for timeouts, tracing, and compression.

## Detailed Explanation

### Core Philosophy
Axum aims to be "the framework for the Tokio ecosystem." It avoids the "magic" of macro-heavy frameworks (like Rocket) in favor of standard Rust syntax and type safety. It uses the `Tower` service trait as its fundamental building block, meaning anything that implements `Service` can be used as a handler or middleware.

### Key Features
*   **Macro-Free API**: Routes and handlers are defined using standard function syntax and method chaining.
*   **Extractor Pattern**: Powerful system to declaratively extract data (JSON, Query Params, Headers) from requests using function arguments.
*   **Tower Compatibility**: Seamless integration with the vast ecosystem of `tower` and `tower-http` middleware.
*   **Tokio Native**: Built directly on Hyper and Tokio, inheriting their performance and asynchronous capabilities.
*   **Type-Safe Routing**: Handlers must match the expected input types, preventing runtime errors.

### Use Cases
*   **Microservices**: Ideal for high-performance, async-heavy services running on Kubernetes.
*   **REST APIs**: Excellent support for JSON (via Serde) and standard HTTP methods.
*   **Middleware-Heavy Apps**: When you need complex request processing pipelines (auth, rate limiting, logging).

### Code Example
*Dependencies: `axum`, `tokio`, `serde`, `serde_json`*

```rust
use axum::{
    routing::{get, post},
    Router,
    Json,
};
use serde::{Deserialize, Serialize};
use std::net::SocketAddr;

#[tokio::main]
async fn main() {
    // Build our application with a route
    let app = Router::new()
        .route("/", get(root))
        .route("/users", post(create_user));

    // Run it with hyper on localhost:3000
    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    println!("listening on {}", addr);
    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}

// Basic handler
async fn root() -> &'static str {
    "Hello, World!"
}

// Handler with JSON extraction and response
#[derive(Deserialize)]
struct CreateUser {
    username: String,
}

#[derive(Serialize)]
struct User {
    id: u64,
    username: String,
}

async fn create_user(Json(payload): Json<CreateUser>) -> Json<User> {
    let user = User {
        id: 1337,
        username: payload.username,
    };
    Json(user)
}
```

## Interview Questions

1.  **Q: How does Axum handle middleware compared to other frameworks?**
    *   **A:** Axum is built on top of the `tower` crate. Middleware in Axum is simply a `tower::Layer` that wraps a `Service`. This allows Axum to use the entire existing ecosystem of Tower middleware (like timeout, tracing, limit) without rewriting them specifically for the framework.

2.  **Q: What is an "Extractor" in Axum?**
    *   **A:** An Extractor is a type that implements the `FromRequest` trait. It allows you to "extract" data from the HTTP request (like the body, headers, or query parameters) simply by adding an argument to your handler function. If extraction fails, Axum automatically returns an appropriate error response (e.g., 400 Bad Request).

3.  **Q: Why might you choose Axum over Rocket?**
    *   **A:** You might choose Axum if you prefer standard Rust syntax over macro magic, if you are already heavily invested in the Tokio/Tower ecosystem, or if you need the absolute latest async features from Tokio. Rocket provides a more "batteries-included" experience but historically had a dependency on nightly Rust (though stable now) and uses more custom syntax.

4.  **Q: Explain the relationship between Axum, Hyper, and Tokio.**
    *   **A:** Tokio is the async runtime that schedules and executes tasks. Hyper is the low-level HTTP implementation that runs on top of Tokio. Axum is the high-level web framework that runs on top of Hyper, providing routing, extraction, and ergonomics.
