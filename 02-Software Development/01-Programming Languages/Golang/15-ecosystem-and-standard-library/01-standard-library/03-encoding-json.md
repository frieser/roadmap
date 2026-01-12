# Encoding/JSON

## Summary
The `encoding/json` package allows Go programs to easily convert between Go data structures and JSON (JavaScript Object Notation). This process is known as **Marshaling** (Go -> JSON) and **Unmarshaling** (JSON -> Go). It relies heavily on **Struct Tags** to map JSON keys to Go struct fields and uses reflection under the hood.

## Detailed Explanation

### 1. Defining Structs
You define the shape of your JSON using Go structs. Exported fields (Capitalized) are required for the package to see them.

```go
type User struct {
    ID        int      `json:"id"`                 // Map "id" to ID
    Name      string   `json:"username"`           // Map "username" to Name
    Email     string   `json:"email,omitempty"`    // Omit if empty string
    Password  string   `json:"-"`                  // Ignore this field completely
    Tags      []string `json:"tags"`
}
```

### 2. Marshaling (Go -> JSON)
Converts a struct (or map/slice) into a JSON byte slice.
```go
u := User{ID: 1, Name: "Alice", Password: "secret"}
jsonData, err := json.Marshal(u)
// jsonData is []byte: {"id":1,"username":"Alice","tags":null}
```

### 3. Unmarshaling (JSON -> Go)
Parses a JSON byte slice into a pointer to a struct.
```go
jsonStr := `{"id": 2, "username": "Bob"}`
var u User
err := json.Unmarshal([]byte(jsonStr), &u)
// u is now User{ID: 2, Name: "Bob"}
```

### 4. Streaming (Encoder/Decoder)
For large payloads or streaming data (like HTTP bodies), use `Encoder` and `Decoder`. They work directly with `io.Reader` and `io.Writer` streams, avoiding the need to load the entire JSON into memory.

```go
// Decode directly from an io.Reader (e.g., http.Request.Body)
err := json.NewDecoder(r.Body).Decode(&u)

// Encode directly to an io.Writer (e.g., http.ResponseWriter)
err := json.NewEncoder(w).Encode(u)
```

### 5. Handling Unknown Structures (`map[string]interface{}`)
If you don't know the JSON structure ahead of time, you can unmarshal into `map[string]interface{}`.
```go
var data map[string]interface{}
json.Unmarshal(bytes, &data)
// Access requires type assertion: data["id"].(float64)
```

## Interview Questions

**Q: Why must struct fields be capitalized for `json.Unmarshal` to work?**
**A:** The `encoding/json` package is in a different package than your code. In Go, unexported fields (lowercase) are private to the package they are defined in. Therefore, `json.Marshal` (which uses reflection) cannot access or modify unexported fields.

**Q: What does the `omitempty` tag option do?**
**A:** It tells the encoder to omit the field from the JSON output if the field has the "zero value" for its type (e.g., `0`, `""`, `false`, `nil`). This is useful for reducing payload size by excluding optional empty fields.

**Q: How does Go handle JSON numbers when decoding into an interface{}?**
**A:** By default, Go decodes valid JSON numbers into `float64` when using `interface{}`. This can cause precision loss for large integers. To avoid this, you can use `json.Decoder` with `.UseNumber()`, which decodes numbers into the `json.Number` type (a string wrapper) that can then be parsed as Int64 or Float64 explicitly.
