#Rust
---
---

## Summary
**Variance** describes how subtyping works with generic arguments (like lifetimes). In Rust, a "longer" lifetime is a subtype of a "shorter" lifetime. **Covariance** allows passing a longer lifetime where a shorter one is expected. **Invariance** forbids this (requiring an exact match), which is crucial for `&mut T` to prevent memory safety violations.

## Detailed Explanation

### 1. Subtyping with Lifetimes
- **Subtype**: A lifetime `'long` is a subtype of `'short` if `'long` lives at least as long as `'short`.
- This seems backwards compared to OOP classes, but it makes sense: if code expects a reference valid for 1 second, giving it one valid for 10 seconds is fine.

### 2. Covariance (`&'a T`)
Shared references are **covariant** over `'a` and `T`.
- If a function expects `&'short str`, you can pass `&'static str`.
- This is why you can pass a `String` (long lived) to a function expecting a temporary slice.

### 3. Invariance (`&'a mut T`)
Mutable references are **invariant** over `T`.
- You generally cannot substitute types inside a mutable reference.
- **Why?** If `&mut T` were covariant, you could overwrite a location expecting a specific subtype with a supertype, leading to reading invalid data later.
- `&'a mut T` is covariant over `'a` (you can borrow a mutable ref for a shorter time), but invariant over `T`.

### 4. Contravariance
Rare in Rust. Occurs in function arguments. `Fn(T)` is contravariant in `T`.
- If you need a function that handles `&'short str`, you can provide a function that handles `&'static str`.

## Rust Application

### The Problem with Mutable Covariance (Invariance)

Imagine if `&mut T` were covariant over `T`:

```rust
fn overwrite<T>(container: &mut T, new_value: T) {
    *container = new_value;
}

fn main() {
    let s: &'static str = "I am static";
    let mut r: &'static str = s;

    {
        let local_string = String::from("I am short lived");
        
        // IF &mut T was covariant, we could pass &mut &'static str 
        // to a function expecting &mut &'short str.
        
        // This would overwrite the static ref 'r' with a pointer to 'local_string'
        // overwrite(&mut r, &local_string); 
    } 
    
    // 'r' would now point to freed memory (local_string)!
    // Rust prevents this by making &mut T INVARIANT.
    // println!("{}", r); 
}
```

### PhantomData for Variance
You can use `PhantomData` to opt-in to variance patterns when building custom smart pointers.

```rust
use std::marker::PhantomData;

struct MyCovariantPtr<'a, T> {
    ptr: *const T,
    _marker: PhantomData<&'a T>, // Opt-in to Covariance
}

struct MyInvariantPtr<'a, T> {
    ptr: *mut T,
    _marker: PhantomData<&'a mut T>, // Opt-in to Invariance
}
```

## Interview Questions

### Q: Why is `&mut T` invariant over `T`?
**A:** To ensure memory safety. If `&mut T` were covariant, you could pass a mutable reference to a specific type (e.g., `&mut &'static str`) to a function expecting a more general type (e.g., `&mut &'a str`). That function could then overwrite the value with a shorter-lived object. When the function returns, the original variable would now hold a dangling pointer to the short-lived object.

### Q: What is the relationship between `'static` and `'a`?
**A:** `'static` is a subtype of every lifetime `'a`. This means `'static` is "longer" or "more useful" than any other lifetime. Anywhere an arbitrary lifetime `'a` is expected, `'static` satisfies that constraint.

### Q: What role does `PhantomData` play in variance?
**A:** `PhantomData<T>` tells the compiler to treat your struct *as if* it contained a `T`, even if it doesn't. This allows you to inherit the variance properties of `T`. For example, `PhantomData<&T>` makes your struct covariant, while `PhantomData<&mut T>` makes it invariant.
