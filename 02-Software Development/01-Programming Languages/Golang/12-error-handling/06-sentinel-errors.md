#Golang
---
---

## Summary

Sentinel Errors are predefined, package-level error variables. They act as unique identifiers for specific error conditions, allowing calling code to check for specific failures using `errors.Is`. They are typically named starting with `Err` (e.g., `io.EOF`, `sql.ErrNoRows`). While simpler than custom error types, they introduce coupling between packages and should be used judiciously.

## Detailed Explanation

### Declaration Pattern

Sentinel errors are usually declared as exported variables using `errors.New`.

```go
package auth

import "errors"

// Naming convention: ErrXxx
var (
    ErrInvalidToken = errors.New("auth: invalid token")
    ErrExpiredToken = errors.New("auth: token expired")
    ErrUserNotFound = errors.New("auth: user not found")
)
```

### Usage Pattern

Callers compare the returned error against these sentinels.

```go
func (s *Service) Login(token string) error {
    if token == "" {
        return auth.ErrInvalidToken // Return the sentinel
    }
    // ...
}

// Caller
err := svc.Login("...")
if errors.Is(err, auth.ErrInvalidToken) {
    // Handle specific case
    return redirectLogin()
}
```

### Why "Sentinel"?

In computer programming, a sentinel value is a special value used to mark a condition or termination (like -1 in a slice index search). Here, the error value itself is the marker. The specific string message inside doesn't matter as much as the *identity* (memory address) of the variable.

### Pros and Cons

**Pros:**
*   Simple to define.
*   Efficient (equality check is fast).
*   Clear API contract ("This function can return ErrX or ErrY").

**Cons:**
*   **Coupling**: The caller must import the package defining the error to check it.
*   **Immutable**: You can't add context (like "user id 123 not found") without wrapping. Wrapping works with `errors.Is`, but the sentinel itself is static.
*   **Public API surface**: Once published, you can't change the error type easily.

### Constant Errors

Since `errors.New` is a function call, sentinels are variables (`var`), which means they are technically mutable (though changing them is bad practice). To make immutable, constant-like errors, you can define a custom type based on string:

```go
type Error string

func (e Error) Error() string { return string(e) }

const ErrConstant = Error("constant error")
```

This is safer but less common than `errors.New`.

## Interview Questions

**Q: What is a Sentinel Error?**

**A:** A Sentinel Error is a predefined, exported global error variable used to indicate a specific error condition (e.g., `io.EOF` or `sql.ErrNoRows`). Callers can check for these errors using `errors.Is` to handle specific failures programmatically without relying on parsing error strings. They serve as unique "markers" in the error flow.

**Q: What is the naming convention for sentinel errors in Go?**

**A:** They should start with `Err` (e.g., `ErrNotFound`, `ErrInvalidInput`). This convention makes them easily distinguishable from other variables and types in documentation and code completion.

**Q: Why might you avoid using Sentinel Errors in a large project?**

**A:** Sentinel errors create a strict dependency between the caller and the defining package. If Package A calls Package B, and checks for `B.ErrFoo`, it must import B. This can lead to import cycles if not managed carefully. Also, sentinel errors carry no data (context). If you need to return "which user was not found", a sentinel `ErrUserNotFound` is insufficient; you'd need a custom error struct or wrapped error.

**Q: How do you handle sentinel errors that have been wrapped?**

**A:** You must use `errors.Is(err, SentinelErr)` instead of `err == SentinelErr`. The equality check only compares the top-level error. `errors.Is` unwraps the error chain to see if the sentinel exists anywhere in the stack.
