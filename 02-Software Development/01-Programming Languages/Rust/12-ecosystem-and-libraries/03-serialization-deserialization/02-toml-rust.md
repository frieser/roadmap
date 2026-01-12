# TOML in Rust
---
---

## Summary
TOML (Tom's Obvious, Minimal Language) is the configuration language of choice for the Rust ecosystem (used in `Cargo.toml`). The `toml` crate provides Serde-compatible serialization and deserialization for TOML files. It is widely used for application configuration due to its readability.

## Detailed Explanation

### Core Philosophy
TOML is designed to be easy for humans to read and write. It maps unambiguously to a hash table. The `toml` crate leverages Serde, meaning if you know how to use Serde with JSON, you already know 90% of how to use it with TOML.

### Key Features
*   **Serde Integration**: Works seamlessly with `#[derive(Serialize, Deserialize)]`.
*   **Preserves Types**: Correctly handles dates, integers, floats, and booleans.
*   **Standard**: Since it's used by Cargo, it is ubiquitous in the Rust world.

### Use Cases
*   **Application Configuration**: Reading `config.toml` files for app settings.
*   **Project Metadata**: Defining build settings or plugin configurations.

### Code Example
*Dependencies: `serde`, `toml`*

```rust
use serde::Deserialize;
use std::collections::HashMap;

#[derive(Deserialize, Debug)]
struct Config {
    package: Package,
    dependencies: HashMap<String, String>,
}

#[derive(Deserialize, Debug)]
struct Package {
    name: String,
    version: String,
}

fn main() {
    let toml_str = r#"
        [package]
        name = "my-app"
        version = "0.1.0"

        [dependencies]
        serde = "1.0"
        toml = "0.8"
    "#;

    let config: Config = toml::from_str(toml_str).unwrap();

    println!("Package Name: {}", config.package.name);
    println!("Dependencies: {:?}", config.dependencies);
}
```

## Interview Questions

1.  **Q: Why is TOML preferred over JSON for configuration files?**
    *   **A:** TOML supports comments (JSON does not), is more readable for humans (less bracket noise), and has a structure that naturally represents key-value pairs and sections, which maps well to configuration hierarchies.

2.  **Q: How do you handle optional fields in a TOML config file using Serde?**
    *   **A:** You wrap the field in `Option<T>` in your struct. If the key is missing in the TOML file, Serde will automatically set the field to `None`. Alternatively, you can use `#[serde(default)]` to populate it with a default value.
