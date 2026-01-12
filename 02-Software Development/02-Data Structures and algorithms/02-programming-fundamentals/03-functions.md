---
---

# Functions

## Summary
Functions are self-contained blocks of code designed to perform a particular task. They promote code reusability, modularity, and maintainability. In Go (Golang), functions are first-class citizens, meaning they can be assigned to variables, passed as arguments, and returned from other functions.

## Detailed Explanation

### 1. Definition and Basics
A function is defined using the `func` keyword, followed by a name, parameters, return types, and a body.
- **Parameters**: Values passed into the function to customize its behavior.
- **Return Values**: Results sent back to the caller.
- **Scope**: Variables defined inside a function are local to that function and cannot be accessed from outside.

### 2. Go (Golang) Specifics

#### Multiple Return Values
Unlike many other languages, Go supports returning multiple values natively. This is commonly used to return both a result and an error.
```go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("cannot divide by zero")
    }
    return a / b, nil
}
```

#### Named Return Values
Return values can be named in the function signature. They act as variables defined at the top of the function. A "naked" `return` statement will return the current values of these variables.
```go
func getCoords() (x, y int) {
    x = 10
    y = 20
    return // returns 10, 20
}
```

#### Variadic Functions
Functions can take an arbitrary number of arguments using the `...` syntax. These arguments are treated as a slice inside the function.
```go
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}
```

#### The `defer` Statement
`defer` schedules a function call to be run immediately before the surrounding function returns. Deferred calls are pushed onto a stack (Last-In-First-Out).
```go
func readData() {
    f, _ := os.Open("file.txt")
    defer f.Close() // Ensures file is closed even if an error occurs
    // ... read file ...
}
```

#### The `init` Function
The `init` function is a special function that takes no arguments and returns nothing. It runs automatically when the package is initialized, before the `main` function.

## Go Code Examples

### Basic Function and Scope
```go
package main

import "fmt"

// Global scope
var greeting = "Hello"

func main() {
    name := "Gopher" // Local scope
    sayHello(name)
}

func sayHello(n string) {
    fmt.Printf("%s, %s!\n", greeting, n)
}
```

### Advanced Features Demo
```go
package main

import "fmt"

// Named return values and multiple returns
func rectangleStats(w, h float64) (area float64, perimeter float64) {
    area = w * h
    perimeter = 2 * (w + h)
    return
}

func main() {
    // Variadic function call
    fmt.Println("Sum:", sum(1, 2, 3, 4, 5))

    a, p := rectangleStats(5, 10)
    fmt.Printf("Area: %.2f, Perimeter: %.2f\n", a, p)

    demoDefer()
}

func sum(nums ...int) int {
    res := 0
    for _, n := range nums {
        res += n
    }
    return res
}

func demoDefer() {
    defer fmt.Println("World")
    fmt.Print("Hello ")
}
```

## Interview Questions

**Q: What are the advantages of named return values in Go?**
**A:** Named return values improve code readability by documenting the meaning of the results in the function signature. They also allow for "naked returns," which can make short functions cleaner, although they should be used sparingly in longer functions to avoid confusion.

**Q: Explain how `defer` works and its execution order.**
**A:** `defer` schedules a function to execute just before the enclosing function returns. If multiple `defer` statements are used, they are executed in **LIFO (Last-In-First-Out)** order. This is particularly useful for resource cleanup (e.g., closing files or unlocking mutexes).

**Q: Can Go functions be passed as arguments?**
**A:** Yes, Go functions are first-class values. You can define a parameter type as a function signature, allowing you to pass functions into other functions (higher-order functions).

**Q: What is a variadic function and how do you pass a slice into one?**
**A:** A variadic function accepts zero or more arguments of a specific type using the `...` syntax. To pass an existing slice into a variadic function, you use the `...` suffix on the slice variable (e.g., `myFunc(mySlice...)`).

**Q: What is the purpose of the `init` function?**
**A:** The `init` function is used for package-level initialization, such as setting up database connections, initializing global variables, or registering drivers. It runs once per package and cannot be called manually.
