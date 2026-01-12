---
---

## Summary
Bash supports one-dimensional indexed arrays. They can hold lists of strings or numbers. Arrays are zero-indexed and allow for sparse population (you can define index 10 without defining index 0-9).

## Detailed Explanation

### Declaration and Assignment
```bash
# Explicit declaration
declare -a my_arr

# Assignment
my_arr=("apple" "banana" "cherry")
my_arr[3]="date"
```

### Accessing Elements
*   **Single Element**: `${my_arr[0]}` (Returns "apple").
*   **All Elements**: `${my_arr[@]}` (Returns all as separate words).
*   **Length**: `${#my_arr[@]}` (Returns count, e.g., 4).
*   **Indices**: `${!my_arr[@]}` (Returns list of defined indices: 0 1 2 3).

### Slicing
*   `${my_arr[@]:1:2}`: Start at index 1, take 2 elements ("banana" "cherry").

### Appending
*   `my_arr+=("elderberry")`

## Go-Specific Context/Examples

Bash arrays are similar to Go **Slices**, but less strict and less performant.

### Analogy
**Bash**:
```bash
arr=("a" "b")
arr+=("c")
echo ${arr[1]}
```
**Go**:
```go
arr := []string{"a", "b"}
arr = append(arr, "c")
fmt.Println(arr[1])
```

## Interview Questions

**Q: What is the difference between `${arr[@]}` and `${arr[*]}`?**
**A:** When quoted:
*   `"${arr[@]}"` expands to distinct arguments: `"a" "b" "c"`. (Preserves spaces within elements). **Always use this.**
*   `"${arr[*]}"` expands to a single string: `"a b c"` (Joined by IFS, usually space).

**Q: How do you delete an element from an array?**
**A:** `unset my_arr[1]`. Note that this does **not** re-index the array. Index 1 is now missing (sparse array). To re-index (compact), you must rebuild it: `my_arr=("${my_arr[@]}")`.

**Q: Can Bash arrays be multi-dimensional?**
**A:** No. Bash only supports 1D arrays. You have to simulate multi-dimensions using associative arrays with keys like `"row,col"`.
