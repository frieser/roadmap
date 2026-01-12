---
---

## Summary
Associative Arrays (dictionaries or hashmaps) allow you to use strings as array indices. They are available in Bash 4.0+. You **must** declare them explicitly using `declare -A`.

## Detailed Explanation

### Declaration
```bash
declare -A user_info
```

### Assignment
```bash
user_info[name]="Alice"
user_info[role]="Admin"
user_info[id]="101"

# Bulk assignment
user_info=([name]="Bob" [role]="Dev")
```

### Accessing
*   **Value**: `${user_info[name]}` (Returns "Alice").
*   **All Values**: `${user_info[@]}`.
*   **All Keys**: `${!user_info[@]}` (Returns "name role id").

### Iterating
```bash
for key in "${!user_info[@]}"; do
    echo "$key -> ${user_info[$key]}"
done
```

## Go-Specific Context/Examples

This is exactly equivalent to Go's **Maps**.

### Analogy
**Bash**:
```bash
declare -A m
m[key]="val"
```
**Go**:
```go
m := make(map[string]string)
m["key"] = "val"
```

## Interview Questions

**Q: What happens if you don't use `declare -A`?**
**A:** Bash treats it as a standard indexed array. `arr[foo]="bar"` will likely be evaluated as `arr[0]="bar"` (since "foo" evaluates to 0 in arithmetic context), overwriting index 0 repeatedly.

**Q: Can you sort an associative array by key?**
**A:** Not natively. You must extract the keys, sort them, and then iterate.
`for k in $(echo ${!arr[@]} | tr ' ' '\n' | sort); do ...`

**Q: Is the order of keys preserved?**
**A:** No. Like Hash Maps in most languages (including Go), iteration order is **random/undefined**. Do not rely on insertion order.
