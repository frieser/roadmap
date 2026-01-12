#Rust
---
---

## Summary
Structs (Structures) are used to create custom data types by grouping related values. Rust has three types of structs: Classic C-style structs (named fields), Tuple structs (unnamed fields), and Unit structs (no fields). Structs are the backbone of data modeling in Rust.

## Detailed Explanation

### 1. Classic Structs
Standard structs with named fields.
```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn build_user(email: String, username: String) -> User {
    User {
        email,      // Field init shorthand
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

### 2. Tuple Structs
Structs with named types but unnamed fields. Useful for creating new types (Newtype pattern) that are distinct from their underlying values.
```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
// black and origin are different types, even if data is same
```

### 3. Unit-Like Structs
Structs with no fields. Useful when you need to implement a trait on a type but don't need to store any data.
```rust
struct AlwaysEqual;
```

### Struct Update Syntax
Create a new instance based on another instance.
```rust
let user2 = User {
    email: String::from("another@example.com"),
    ..user1 // Copy remaining fields from user1
};
```
*Note*: If fields implementing `Copy` are moved, `user1` might become partially invalid.

## Interview Questions

1. **What is the "Field Init Shorthand"?**
   - If a variable name matches the field name in the struct definition, you can write `field_name` instead of `field_name: field_name` when initializing the struct.

2. **What is a Tuple Struct and when would you use it?**
   - It is a struct without named fields. It is commonly used for the "Newtype Pattern" to enforce type safety (e.g., distinguishing `Meters(u32)` from `Miles(u32)`) without the overhead of named fields.

3. **What happens to ownership when using struct update syntax?**
   - Fields that are not explicitly set are moved (or copied) from the base instance. If a field implementing only `Move` (like String) is moved into the new struct, the original struct can no longer be used.
