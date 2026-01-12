# Loco
---
---

## Summary
Loco is a "batteries-included" web framework for Rust, heavily inspired by Ruby on Rails. While frameworks like Axum and Actix are minimalist and require you to assemble your own stack, Loco provides a cohesive, opinionated structure with built-in tools for databases, authentication, background workers, and mailers. It aims to be the "one-person framework" for shipping products fast.

## Detailed Explanation

### Core Philosophy
**"Convention over Configuration."** Loco believes developers shouldn't waste time deciding folder structures or wiring up libraries. It provides a standard way to build apps so you can focus on business logic. It leverages `Axum` for the web layer and `SeaORM` for the database layer.

### Key Features
*   **Scaffolding**: CLI tools (`cargo loco generate`) to create models, controllers, and tests instantly.
*   **MVC Architecture**: Enforces a Model-View-Controller structure.
*   **Built-in Auth**: Standard JWT and session authentication out of the box.
*   **Background Workers**: Integrated task queue implementation (like Sidekiq) for async jobs.
*   **SeaORM Integration**: Strong async ORM support with migrations and entities.

### Use Cases
*   **Startup MVPs**: When speed of delivery is critical and you need standard features (auth, db) immediately.
*   **Enterprise Applications**: Where standard structure helps onboarding new developers.
*   **REST APIs**: Rapidly building CRUD APIs with auto-generated code.

### Code Example
*Dependencies: `loco-rs`*

```rust
// src/controllers/user.rs
use loco_rs::prelude::*;
use crate::models::_entities::users;

// Standard handler receiving the AppContext
pub async fn list_users(State(ctx): State<AppContext>) -> Result<Response> {
    // SeaORM syntax to fetch all users
    let users = users::Entity::find().all(&ctx.db).await?;
    format::json(users)
}

pub async fn create_user(
    State(ctx): State<AppContext>,
    Json(params): Json<users::ActiveModel>,
) -> Result<Response> {
    // Insert into DB using ActiveModel
    let user = params.insert(&ctx.db).await?;
    format::json(user)
}

// Route definition
pub fn routes() -> Routes {
    Routes::new()
        .prefix("api/users")
        .add("/", get(list_users))
        .add("/", post(create_user))
}
```

## Interview Questions

1.  **Q: What libraries does Loco use under the hood?**
    *   **A:** Loco is built primarily on top of **Axum** (for the web server/HTTP handling) and **SeaORM** (for database interactions). It adds layers for configuration, task processing, and mailers to glue these together into a Rails-like experience.

2.  **Q: How does Loco handle database migrations?**
    *   **A:** Loco uses SeaORM's migration toolkit. You generate migrations via the CLI (`cargo loco generate migration`), which creates a Rust file where you define the schema changes using a fluent API. Migrations are compiled into the binary and run via a command like `cargo loco db migrate`.

3.  **Q: What is the benefit of using Loco over raw Axum?**
    *   **A:** Raw Axum gives you a router and tools to build handlers, but you have to decide how to structure your folders, which ORM to pick, how to handle configuration, logging, and background tasks. Loco makes all these decisions for you, saving setup time and enforcing a consistent architecture that is familiar to developers coming from Rails or Laravel.
