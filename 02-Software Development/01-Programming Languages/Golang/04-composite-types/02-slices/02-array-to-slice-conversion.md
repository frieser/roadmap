#Golang
---
---

## Summary

Converting an array to a slice in Go creates a slice header that references the original array's memory—no data is copied. This means modifications through the slice affect the original array. The syntax `array[:]` creates a full slice, while `array[low:high]` creates a partial slice. Understanding this shared memory relationship is crucial for avoiding unintended side effects.

## Detailed Explanation

### **Basic Conversion Syntax**

```go
package main

import "fmt"

func main() {
    arr := [5]int{1, 2, 3, 4, 5}
    
    // Full slice (all elements)
    slice := arr[:]
    fmt.Println(slice)  // [1 2 3 4 5]
    
    // Partial slice
    partial := arr[1:4]
    fmt.Println(partial)  // [2 3 4]
    
    // From start
    fromStart := arr[:3]
    fmt.Println(fromStart)  // [1 2 3]
    
    // To end
    toEnd := arr[2:]
    fmt.Println(toEnd)  // [3 4 5]
}
```

### **Shared Memory (No Copy)**

```go
func main() {
    arr := [5]int{1, 2, 3, 4, 5}
    slice := arr[:]
    
    // Modifying slice modifies array
    slice[0] = 100
    fmt.Println(arr)    // [100 2 3 4 5] (modified!)
    fmt.Println(slice)  // [100 2 3 4 5]
    
    // Modifying array modifies slice
    arr[1] = 200
    fmt.Println(arr)    // [100 200 3 4 5]
    fmt.Println(slice)  // [100 200 3 4 5]
}
```

### **Slice Capacity from Array**

```go
func main() {
    arr := [10]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
    
    // Full slice
    s1 := arr[:]
    fmt.Printf("s1: len=%d cap=%d\n", len(s1), cap(s1))
    // s1: len=10 cap=10
    
    // Slice from middle
    s2 := arr[3:7]
    fmt.Printf("s2: len=%d cap=%d %v\n", len(s2), cap(s2), s2)
    // s2: len=4 cap=7 [3 4 5 6]
    // cap=7 because slice can grow to end of array (10-3=7)
    
    // Slice near end
    s3 := arr[8:]
    fmt.Printf("s3: len=%d cap=%d %v\n", len(s3), cap(s3), s3)
    // s3: len=2 cap=2 [8 9]
}
```

### **Three-Index Slice for Capacity Control**

```go
func main() {
    arr := [10]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
    
    // Normal slice: can append into array's remaining space
    s1 := arr[2:5]
    fmt.Printf("s1: len=%d cap=%d\n", len(s1), cap(s1))
    // len=3, cap=8
    
    s1 = append(s1, 100)  // Overwrites arr[5]!
    fmt.Println(arr)  // [0 1 2 3 4 100 6 7 8 9]
    
    // Reset
    arr = [10]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
    
    // Three-index slice: limit capacity
    s2 := arr[2:5:5]  // cap = 5-2 = 3
    fmt.Printf("s2: len=%d cap=%d\n", len(s2), cap(s2))
    // len=3, cap=3
    
    s2 = append(s2, 100)  // Creates NEW array (exceeds cap)
    fmt.Println(arr)  // [0 1 2 3 4 5 6 7 8 9] (unchanged!)
    fmt.Println(s2)   // [2 3 4 100]
}
```

### **Passing Arrays to Slice-Taking Functions**

```go
// Function that takes a slice
func sum(nums []int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

func main() {
    arr := [5]int{1, 2, 3, 4, 5}
    
    // Pass array as slice
    result := sum(arr[:])
    fmt.Println(result)  // 15
    
    // Pass partial array
    result = sum(arr[1:4])
    fmt.Println(result)  // 9 (2+3+4)
}
```

### **Function Modifying Array via Slice**

```go
func double(nums []int) {
    for i := range nums {
        nums[i] *= 2
    }
}

func main() {
    arr := [5]int{1, 2, 3, 4, 5}
    
    // Slice shares memory with array
    double(arr[:])
    
    fmt.Println(arr)  // [2 4 6 8 10] (modified!)
}
```

### **Creating Independent Copy**

```go
func main() {
    arr := [5]int{1, 2, 3, 4, 5}
    
    // Method 1: make + copy
    slice := make([]int, len(arr))
    copy(slice, arr[:])
    
    // Method 2: append to nil slice
    slice2 := append([]int(nil), arr[:]...)
    
    // Now modifications are independent
    slice[0] = 100
    slice2[1] = 200
    
    fmt.Println(arr)     // [1 2 3 4 5] (unchanged)
    fmt.Println(slice)   // [100 2 3 4 5]
    fmt.Println(slice2)  // [1 200 3 4 5]
}
```

### **Use Case: Working with Fixed Buffers**

```go
func processPacket(packet *[1024]byte) int {
    // Create slice view for easier manipulation
    data := packet[:]
    
    // Find header end
    headerEnd := bytes.IndexByte(data, '\n')
    if headerEnd == -1 {
        return 0
    }
    
    header := data[:headerEnd]
    body := data[headerEnd+1:]
    
    // Process header and body...
    return len(header) + len(body)
}
```

### **Use Case: Stack-Allocated Working Memory**

```go
func hashFile(path string) ([]byte, error) {
    // Stack-allocated buffer (array)
    var buf [4096]byte
    
    file, err := os.Open(path)
    if err != nil {
        return nil, err
    }
    defer file.Close()
    
    h := sha256.New()
    for {
        // Use slice of array for Read
        n, err := file.Read(buf[:])
        if err == io.EOF {
            break
        }
        if err != nil {
            return nil, err
        }
        h.Write(buf[:n])
    }
    
    return h.Sum(nil), nil
}
```

### **Comparison Table**

| Operation | Result | Memory |
| --- | --- | --- |
| `arr[:]` | Full slice of array | Shared |
| `arr[low:high]` | Partial slice | Shared |
| `arr[low:high:max]` | Capacity-limited slice | Shared (until append exceeds cap) |
| `append([]T(nil), arr[:]...)` | Independent copy | New allocation |
| `make([]T, len(arr)); copy(...)` | Independent copy | New allocation |

### **Common Gotchas**

```go
func main() {
    // Gotcha 1: Function receives copy of array, not reference
    func modify(arr [3]int) {
        arr[0] = 100  // Modifies copy, not original!
    }
    
    original := [3]int{1, 2, 3}
    modify(original)
    fmt.Println(original)  // [1 2 3] (unchanged)
    
    // Fix: Pass pointer or slice
    func modifySlice(s []int) {
        s[0] = 100  // Modifies original via slice
    }
    modifySlice(original[:])
    fmt.Println(original)  // [100 2 3]
    
    // Gotcha 2: Slice from subslice can still access original array
    arr := [10]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
    sub := arr[2:5]
    sub = sub[:cap(sub)]  // Extend to capacity
    fmt.Println(sub)  // [2 3 4 5 6 7 8 9] (sees beyond original slice!)
}
```

## Interview Questions

**Q: Does converting an array to a slice copy the data?**
**A:** No. `arr[:]` creates a slice header pointing to the array's memory—no data is copied. Modifications through the slice affect the original array. To create an independent copy, use `copy()` or `append([]T(nil), arr[:]...)`.

**Q: What is the capacity of a slice created from an array?**
**A:** The capacity is the number of elements from the slice's start position to the end of the underlying array. For `arr[3:7]` on a 10-element array, capacity is 7 (from index 3 to 9). Use three-index slicing `arr[3:7:7]` to limit capacity.

**Q: Why would you use a three-index slice when converting from an array?**
**A:** To prevent `append` from overwriting array elements beyond the slice. With `arr[2:5:5]`, capacity equals length (3), so appending forces a new allocation instead of writing into `arr[5]`. This protects the original array from unintended modification.

**Q: How do you pass an array to a function that expects a slice?**
**A:** Use `arr[:]` to create a full slice, or `arr[low:high]` for a partial slice. Note that modifying the slice in the function will modify the original array. If you need the function to work with a copy, create one first with `copy()`.
