#Rust
---
---

## Summary
Pattern matching is a core feature of Rust, primarily via the `match` expression. It allows you to compare a value against a series of patterns and execute code based on which pattern matches. It is exhaustive, meaning you must handle every possible case. `if let` provides a concise way to handle a single pattern.

## Detailed Explanation

### The `match` Expression
Similar to `switch` in other languages, but much more powerful.
- **Exhaustiveness**: The compiler forces you to handle all possibilities (e.g., all enum variants).
- **Binding**: Patterns can bind parts of the values to variables.

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(String), // Variant with data
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {}!", state);
            25
        },
    }
}
```

### Catch-all and Placeholders
- `_` : The catch-all pattern (matches anything, does not bind).
- `other` : Binds the value to the variable `other`.

```rust
let dice_roll = 9;
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    _ => reroll(), // Catch-all
}
```

### `if let`
Syntax sugar for a `match` that runs code only when the value matches one pattern and ignores the rest.
```rust
let config_max = Some(3u8);

// Verbose match
match config_max {
    Some(max) => println!("The maximum is configured to be {}", max),
    _ => (),
}

// Concise if let
if let Some(max) = config_max {
    println!("The maximum is configured to be {}", max);
}
```

### Destructuring
You can break apart structs, enums, tuples, and references.
```rust
struct Point { x: i32, y: i32 }
let p = Point { x: 0, y: 7 };
let Point { x, y } = p; // Creates variables x and y
```

## Interview Questions

1. **What does it mean for `match` to be exhaustive?**
   - It means you must account for every possible value of the type being matched. If you miss a case, the code will not compile. This prevents bugs where new enum variants are added but not handled.

2. **When should you use `if let` over `match`?**
   - Use `if let` when you only care about one specific pattern and want to ignore all others (the `_` case). Use `match` when you need to handle multiple distinct cases or verify exhaustiveness.

3. **Can patterns shadow variables?**
   - Yes. Variables introduced inside a match arm (e.g., `Some(x)`) shadow variables of the same name from the outer scope within that arm's block.
