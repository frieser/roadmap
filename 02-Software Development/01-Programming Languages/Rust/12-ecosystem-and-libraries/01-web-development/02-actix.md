# Actix Web
---
---

## Summary
Actix Web is a powerful, pragmatic, and extremely fast web framework for Rust. It is based on the Actor model (though the actor parts are now largely abstracted away in recent versions) and is renowned for consistently topping the TechEmpower benchmarks for performance. It provides a mature, feature-rich environment for building scalable web applications.

## Detailed Explanation

### Core Philosophy
Actix Web prioritizes performance and reliability. Originally built strictly on the Actor model (via the `actix` crate), modern versions (v4+) use a simplified execution model while retaining the performance benefits. It aims to provide a robust foundation where speed is a default, not an afterthought.

### Key Features
*   **Extreme Performance**: Consistently one of the fastest web frameworks in any language.
*   **Actor System Roots**: While less visible now, the architecture handles concurrency exceptionally well.
*   **Type-Safe Request Information**: Strongly typed extractors for forms, JSON, and path parameters.
*   **Rich Ecosystem**: Mature middleware support, distinct from Tower but equally capable.
*   **HTTP/2 and WebSocket Support**: First-class support for modern web protocols.

### Use Cases
*   **High-Load Systems**: Ad-tech, gaming backends, or any service handling millions of requests per second.
*   **Microservices**: Robust enough for complex distributed systems.
*   **Real-time Applications**: Excellent WebSocket integration for chat apps or live feeds.

### Code Example
*Dependencies: `actix-web`*

```rust
use actix_web::{get, post, web, App, HttpResponse, HttpServer, Responder};

#[get("/")]
async fn hello() -> impl Responder {
    HttpResponse::Ok().body("Hello world!")
}

#[post("/echo")]
async fn echo(req_body: String) -> impl Responder {
    HttpResponse::Ok().body(req_body)
}

async fn manual_hello() -> impl Responder {
    HttpResponse::Ok().body("Hey there!")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(hello)
            .service(echo)
            .route("/hey", web::get().to(manual_hello))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

## Interview Questions

1.  **Q: What is the main architectural difference between Actix Web and frameworks like Axum?**
    *   **A:** Actix Web implements its own service traits and runtime logic (though it runs on Tokio), whereas Axum is tightly coupled to the standard `tower::Service` ecosystem. Actix historically used the Actor model for everything, which influenced its design to be highly concurrent and isolated.

2.  **Q: How does Actix Web achieve its high performance?**
    *   **A:** It minimizes allocations, uses highly optimized HTTP parsers, and effectively leverages Rust's zero-cost abstractions. Its architecture allows it to scale linearly with the number of CPU cores by spawning a separate thread with its own runtime for handling requests (thread-local architecture).

3.  **Q: What is `App::data` vs `App::app_data` in Actix Web?**
    *   **A:** `App::data` (deprecated/legacy in some contexts) wraps data in an `Arc` internally. `App::app_data` is the raw configuration method; typically, you use `web::Data::new()` (which is an `Arc` wrapper) and pass it to `.app_data()` to share state across the application threads.

4.  **Q: Why does Actix Web start multiple worker threads by default?**
    *   **A:** Actix Web follows a "share nothing" architecture where it spawns a number of workers equal to the number of logical CPU cores. Each worker runs its own instance of the application (and its own Tokio runtime), ensuring that processing is evenly distributed and no single lock contends across all cores.
