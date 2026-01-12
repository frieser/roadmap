---
---

## Summary
In Bash, variables defined inside a function are **global** by default. This is a common source of bugs where a function accidentally overwrites a variable in the main script. To prevent this, you must explicitly use the `local` keyword.

## Detailed Explanation

### Global (Default)
```bash
my_func() {
    var="I am global"
}
my_func
echo $var  # Output: "I am global"
```

### Local
```bash
my_func() {
    local var="I am local"
}
my_func
echo $var  # Output: (Empty or previous value)
```

### Return Values
Bash functions strictly return an **exit status** (0-255). They cannot return strings or arrays like Go functions. To "return" data, you `echo` it to stdout and capture it:
`result=$(my_func)`

## Go-Specific Context/Examples

Go has strict lexical scoping (block scope).

### Analogy
*   **Bash `local`**: Similar to defining a variable inside a Go function `func() { x := 1 }`.
*   **Bash Default**: Similar to assigning to a global variable `var x; func() { x = 1 }`.

## Interview Questions

**Q: What happens if you declare `local` outside a function?**
**A:** It causes an error: `local: can only be used in a function`.

**Q: Can a local variable shadow a global variable?**
**A:** Yes. If you have `x=1` global and `local x=2` inside a function, the function sees 2. The global `x` remains 1 after the function exits. This is generally good practice to avoid side effects.

**Q: How do you return a boolean from a function?**
**A:** Use `return 0` for True and `return 1` for False. Then check it with `if my_func; then ...`.
