#Rust
---
---

## Summary
Rust's control flow is expression-based. `if` blocks return values, and loops (`loop`, `while`, `for`) are powerful constructs that can also return values. The language favors iterators over C-style `for` loops for safety and performance.

## Detailed Explanation

### If Expressions
`if` is an expression, not a statement. This means it can return a value, which can be assigned to a variable.
- All arms of the `if` must return the same type.
- Since it's an expression, you don't need ternary operators (Rust doesn't have `? :`).

```rust
let condition = true;
let number = if condition { 5 } else { 6 };
```

### Loops

#### 1. `loop`
An infinite loop. It is the primitive underlying `while` and `for`.
- Can return a value using `break value`.
- Can break outer loops using labels (`'label`).

```rust
let result = loop {
    counter += 1;
    if counter == 10 {
        break counter * 2; // Returns 20
    }
};
```

#### 2. `while`
Runs while a condition is true. Standard C-style behavior.
```rust
while number != 0 {
    println!("{}", number);
    number -= 1;
}
```

#### 3. `for`
The most common loop in Rust. It iterates over a collection (anything implementing `IntoIterator`).
- Safer than C-style loops (no off-by-one errors).
- Faster (compiler can optimize bounds checks).

```rust
let a = [10, 20, 30, 40, 50];

for element in a {
    println!("the value is: {}", element);
}

// Range syntax
for number in (1..4).rev() {
    println!("{}!", number);
}
```

## Interview Questions

1. **Why is `if` called an expression in Rust?**
   - Because it evaluates to a value. This allows assigning the result of an `if-else` block directly to a variable, eliminating the need for a ternary operator.

2. **How do you return a value from a loop?**
   - By placing the value after the `break` keyword: `break my_value;`. This is only valid for `loop`, not `while` or `for`.

3. **Why does Rust prefer `for` loops over `while` loops for iterating arrays?**
   - `for` loops are safer because they eliminate the risk of invalid index access (bounds checks are handled by the iterator). They are often faster because the compiler can elide runtime bounds checks that might be necessary in a `while` loop.
