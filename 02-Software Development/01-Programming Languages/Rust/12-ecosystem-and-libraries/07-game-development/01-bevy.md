# Bevy
---
---

## Summary
Bevy is a modern, data-driven game engine built in Rust. It is currently the most popular and active engine in the Rust ecosystem. Its distinguishing feature is its **Entity Component System (ECS)** architecture, which is built into the core of the engine, not just an add-on. It is simple, modular, and highly parallel.

## Detailed Explanation

### Core Philosophy
Bevy is "ECS-first." Everything in Bevy is an Entity, a Component, or a System. This allows the engine to automatically parallelize game logic across all available CPU cores without the developer needing to manage threads manually. It aims to be simple to learn for beginners but scalable for complex games.

### Key Features
*   **ECS Architecture**: Highly performant, ergonomic ECS.
*   **Parallelism**: Systems run in parallel by default.
*   **Hot Reloading**: Fast iteration times for assets and systems.
*   **Render Graph**: Flexible, modular rendering pipeline (based on wgpu).
*   **Plugin System**: The engine itself is just a collection of plugins. You can disable what you don't need.

### Use Cases
*   **Indie Games**: 2D and 3D games.
*   **Simulation**: High-performance simulations with thousands of entities.
*   **Data Visualization**: Using the ECS to manage complex data states.

### Code Example
*Dependencies: `bevy`*

```rust
use bevy::prelude::*;

#[derive(Component)]
struct Person {
    name: String,
}

#[derive(Component)]
struct Greeter;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, setup)
        .add_systems(Update, greet_people)
        .run();
}

fn setup(mut commands: Commands) {
    commands.spawn(Person {
        name: "Elaina".to_string(),
    });
    commands.spawn(Person {
        name: "Ren".to_string(),
    });
}

fn greet_people(query: Query<&Person>) {
    for person in &query {
        println!("hello {}!", person.name);
    }
}
```

## Interview Questions

1.  **Q: What is an ECS and why does Bevy use it?**
    *   **A:** ECS (Entity Component System) is a pattern where data (Components) is separated from logic (Systems). Entities are just IDs that group components. Bevy uses it because it creates a memory layout that is cache-friendly (contiguous arrays of data) and allows for easy parallel execution of systems that don't access the same data.

2.  **Q: How does Bevy handle Concurrency?**
    *   **A:** Bevy's scheduler analyzes the systems and their data access requirements (Queries). If System A reads Component X and System B reads Component Y, they are run in parallel. If System A writes to X, and System C reads X, they are ordered correctly. This is done automatically.

3.  **Q: What are "Bundles" in Bevy?**
    *   **A:** A Bundle is a template for creating entities. It is a collection of components that are commonly added together (e.g., a `SpriteBundle` contains a `Transform`, `GlobalTransform`, `Sprite`, `Handle<Image>`, etc.). It makes spawning entities ergonomic.
