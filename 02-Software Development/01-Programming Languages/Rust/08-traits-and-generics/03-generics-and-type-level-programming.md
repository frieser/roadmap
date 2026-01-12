#Rust
---
---

## Summary
Generics allow you to write flexible, reusable code that works with any data type. Rust implements generics using **Monomorphization**, meaning the compiler generates specialized code for each concrete type used, resulting in zero runtime cost. Advanced features include **Const Generics** (values as parameters), **PhantomData** (for type safety), and **Marker Traits** (`Send`, `Sync`) for concurrency guarantees.

## Detailed Explanation

### 1. Monomorphization
When you compile generic code, Rust identifies every concrete type used in place of `<T>` and generates a specific copy of the function or struct for that type.
- **Pro**: No runtime overhead (static dispatch). Code is as fast as hand-written specific versions.
- **Con**: Increases binary size ("code bloat") and compile times.

### 2. Const Generics
Generics aren't limited to types; they can also accept specific values.
- Syntax: `<T, const N: usize>`
- Usage: Commonly used for array sizes (e.g., `[T; N]`) or mathematical operations where dimensions must be checked at compile time.

### 3. PhantomData
`std::marker::PhantomData<T>` is a zero-sized type used to "pretend" a struct owns a `T`, even if it doesn't store it.
- **Use Case**: Telling the compiler that your struct logically acts like it owns a `T` (for drop check or lifetime variance) or to carry type information for state machines.

### 4. Marker Traits
Traits with no methods, used to communicate properties to the compiler.
- **`Send`**: Types that can be safely transferred across thread boundaries.
- **`Sync`**: Types that can be safely referenced (`&T`) from multiple threads.
- **`Copy`**: Types that can be duplicated by simply copying bits (memcpy).

## Rust Application

### Basic Generics and Monomorphization

```rust
// Generic Struct
struct Point<T> {
    x: T,
    y: T,
}

// Generic Method
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let int_point = Point { x: 5, y: 10 };
    let float_point = Point { x: 1.0, y: 4.0 };
    
    // Compiler generates:
    // struct Point_i32 { x: i32, y: i32 }
    // struct Point_f64 { x: f64, y: f64 }
}
```

### Const Generics (Arrays)

```rust
struct Buffer<T, const SIZE: usize> {
    data: [T; SIZE],
}

impl<T: Default + Copy, const SIZE: usize> Buffer<T, SIZE> {
    fn new() -> Self {
        Self {
            data: [T::default(); SIZE],
        }
    }
}

fn main() {
    // Valid at compile time
    let b1 = Buffer::<i32, 10>::new(); 
    let b2 = Buffer::<f64, 50>::new();
}
```

### PhantomData for State

```rust
use std::marker::PhantomData;

struct User<State> {
    name: String,
    state: PhantomData<State>, // Takes 0 bytes
}

struct Active;
struct Suspended;

fn main() {
    // We can distinguish these types at compile time!
    let active_user: User<Active> = User { 
        name: "Alice".into(), 
        state: PhantomData 
    };
    
    // let suspended_user: User<Suspended> = active_user; // ERROR: Type mismatch
}
```

## Interview Questions

### Q: What is Monomorphization and what is its trade-off?
**A:** Monomorphization is the process where the compiler generates a unique copy of a generic function or struct for every concrete type combination used. The **benefit** is zero-cost abstraction (no runtime overhead, maximum optimization). The **trade-off** is increased binary size and potentially slower compilation times.

### Q: Why do we need `PhantomData`?
**A:** Since Rust's compiler is smart, it drops unused generic parameters. If you define `struct MyStruct<T>`, but don't use `T` in a field, the compiler errors. `PhantomData<T>` allows you to "use" `T` without adding any runtime size (it's 0 bytes). It's crucial for instructing the compiler about drop behavior, lifetime variance, or for type-state programming patterns.

### Q: What are `Send` and `Sync` traits?
**A:** They are "marker traits" (traits with no methods) that enforce thread safety.
- `Send`: Ownership of the type can be moved to another thread.
- `Sync`: It is safe to access the type via a shared reference (`&T`) from multiple threads (equivalent to `T` is `Send`).
Most primitive types implement these automatically, but raw pointers (`*const T`) do not, requiring manual implementation (`unsafe impl`) if you're building low-level concurrency primitives.
