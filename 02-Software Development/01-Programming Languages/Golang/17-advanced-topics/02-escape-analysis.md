# Escape Analysis

## Summary
Escape Analysis is the process the Go compiler uses to determine whether a variable should be allocated on the **stack** or the **heap**. If the compiler can prove a variable is not referenced outside the function it's defined in, it stays on the stack (fast). If it "escapes" to other functions or global scope, it moves to the heap (GC managed).

## Detailed Explanation

### How It Works
During the compilation phase, Go builds a call graph of the code. It traces the flow of pointers and values.
*   **Does not escape**: The variable's lifetime is fully contained within the function stack frame.
*   **Escapes**: The variable is referenced after the function returns (e.g., returning a pointer) or the compiler cannot prove its size or lifetime (e.g., interfaces).

### Common Escape Scenarios
1.  **Returning Pointers**: Returning `&variable` forces `variable` to the heap because it must survive the function return.
2.  **Interface Assignments**: Storing a value in an `interface{}` often escapes because the compiler may not know the concrete type's size or method implementation at compile time.
3.  **Closures**: Variables captured by closures may escape if the closure itself escapes.
4.  **Large Slices/Maps**: If a stack frame is too big (stack size limit), large objects are moved to the heap.
5.  **`fmt.Println`**: Arguments to `fmt` functions almost always escape because they accept `interface{}`.

### Checking Escape Analysis
You can see exactly what the compiler is deciding by using the `-gcflags` flag:

```bash
go build -gcflags '-m' main.go
# -m prints optimization decisions
# -m -m prints deeper details
```

### Code Example

```go
package main

type Data struct {
    x int
}

// Global variable (always heap)
var global *Data

func main() {
    // Case 1: Stack
    // 'a' is never passed out.
    a := Data{x: 1} 
    _ = a.x

    // Case 2: Heap (escapes via return)
    b := createData() 
    
    // Case 3: Heap (escapes to global)
    c := Data{x: 3}
    global = &c 
    
    // Case 4: Heap (interface{})
    d := Data{x: 4}
    doSomething(d)
}

func createData() *Data {
    d := Data{x: 2}
    return &d // Escapes: address returned
}

func doSomething(v interface{}) {
    // usage of interface causes escape depending on impl
}
```

## Interview Questions

**Q: What is the command to check escape analysis?**
**A:** `go build -gcflags '-m'`.

**Q: Why is stack allocation preferred over heap?**
**A:** Stack allocation is effectively free (just moving the stack pointer) and automatically cleaned up when the function returns. Heap allocation requires searching for free memory and adds pressure to the Garbage Collector.

**Q: Does using an interface always cause an escape?**
**A:** Not always, but very often. If the compiler can de-virtualize the call or prove the interface value doesn't escape, it might stay on stack. However, dynamic dispatch usually hinders this analysis.
