# Rocket
---
---

## Summary
Rocket is a web framework for Rust that focuses on usability, security, and type safety. Known for its beautiful, macro-driven API, it was one of the first major web frameworks in the Rust ecosystem. It aims to make writing web applications simple and intuitive without sacrificing flexibility or performance.

## Detailed Explanation

### Core Philosophy
Rocket's philosophy is to provide a "premium" developer experience. It relies heavily on Rust's procedural macros to handle routing, request parsing, and validation declaratively. This results in clean, readable code where the intent is obvious. It also enforces strict type safety: if a handler requests a `User` struct from a JSON body, the route won't even match unless the JSON is valid and parseable.

### Key Features
*   **Attribute-Based Routing**: Define routes using decorators like `#[get("/path")]`.
*   **Request Guards**: A powerful system to validate and retrieve data (API keys, database connections) before the handler is called.
*   **Type-Safe URI Generation**: Generate URLs for routes in your code, checked at compile time.
*   **Built-in Testing**: First-class support for local integration testing without binding to a network port.
*   **Managed State**: Easy dependency injection for database pools and configuration.

### Use Cases
*   **Public APIs**: Where type safety and input validation are paramount.
*   **Monolithic Applications**: Its rich feature set supports complex applications well.
*   **Beginners**: Its intuitive API is often considered easier to learn for those new to Rust web dev.

### Code Example
*Dependencies: `rocket`*

```rust
#[macro_use] extern crate rocket;

use rocket::serde::{Deserialize, json::Json};

#[derive(Deserialize)]
#[serde(crate = "rocket::serde")]
struct Task {
    description: String,
    complete: bool,
}

#[get("/")]
fn index() -> &'static str {
    "Hello, world!"
}

#[post("/task", format = "json", data = "<task>")]
fn new_task(task: Json<Task>) -> String {
    format!("Created task: {}", task.description)
}

// Request Guard Example: extracting a user ID from a header (simplified)
struct ApiKey(String);

#[launch]
fn rocket() -> _ {
    rocket::build()
        .mount("/", routes![index, new_task])
}
```

## Interview Questions

1.  **Q: How do Request Guards in Rocket differ from Middleware?**
    *   **A:** Middleware typically wraps the entire request processing pipeline (logging, compression). Request Guards are specific to a route handler and run *before* the handler. They can pass data into the handler (like an authenticated user) or reject the request entirely (forwarding it to the next matching route) if the guard fails.

2.  **Q: Rocket historically required Nightly Rust. Is this still true?**
    *   **A:** No. Since version 0.5, Rocket works on **Stable Rust**. The framework was refactored to use stable procedural macros and async features, removing the long-standing requirement for the nightly compiler.

3.  **Q: Explain the concept of "Fairings" in Rocket.**
    *   **A:** Fairings are Rocket's take on structured middleware. Unlike generic middleware, Fairings hook into specific lifecycle events: `on_ignite` (startup), `on_liftoff` (server start), `on_request`, `on_response`, and `on_shutdown`. They are commonly used for tasks like database migrations, configuration setup, or modifying responses globally.

4.  **Q: How does Rocket handle blocking I/O?**
    *   **A:** Rocket 0.5 is fully asynchronous (based on Tokio). However, if you need to perform blocking operations (like heavy CPU tasks or legacy synchronous I/O), you should use `tokio::task::spawn_blocking` or similar mechanisms to avoid blocking the async runtime loop.
