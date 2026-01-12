#Golang
---
---

## Summary

Struct tags are metadata strings attached to struct fields that provide instructions to packages like `encoding/json`, `encoding/xml`, and ORMs. For JSON specifically, tags control field names, omit empty values, and handle special cases. The `encoding/json` package uses these tags during marshaling (Go → JSON) and unmarshaling (JSON → Go) to produce idiomatic JSON output and parse incoming data correctly.

## Detailed Explanation

### **Basic Struct Tag Syntax**

```go
type User struct {
    FieldName Type `key:"value" key2:"value2"`
}
```

Tags are raw string literals (backticks) following the field type:

```go
type User struct {
    ID        int    `json:"id"`
    FirstName string `json:"first_name"`
    Email     string `json:"email"`
}
```

### **JSON Marshaling (Go → JSON)**

```go
package main

import (
    "encoding/json"
    "fmt"
)

type User struct {
    ID        int    `json:"id"`
    FirstName string `json:"first_name"`
    LastName  string `json:"last_name"`
    Email     string `json:"email"`
}

func main() {
    user := User{
        ID:        1,
        FirstName: "John",
        LastName:  "Doe",
        Email:     "john@example.com",
    }
    
    // Marshal to JSON
    data, err := json.Marshal(user)
    if err != nil {
        panic(err)
    }
    
    fmt.Println(string(data))
    // {"id":1,"first_name":"John","last_name":"Doe","email":"john@example.com"}
    
    // Pretty print
    data, _ = json.MarshalIndent(user, "", "  ")
    fmt.Println(string(data))
    // {
    //   "id": 1,
    //   "first_name": "John",
    //   "last_name": "Doe",
    //   "email": "john@example.com"
    // }
}
```

### **JSON Unmarshaling (JSON → Go)**

```go
func main() {
    jsonData := `{
        "id": 1,
        "first_name": "Jane",
        "last_name": "Smith",
        "email": "jane@example.com"
    }`
    
    var user User
    err := json.Unmarshal([]byte(jsonData), &user)
    if err != nil {
        panic(err)
    }
    
    fmt.Printf("%+v\n", user)
    // {ID:1 FirstName:Jane LastName:Smith Email:jane@example.com}
}
```

### **Common JSON Tag Options**

| Tag | Effect | Example |
| --- | --- | --- |
| `json:"name"` | Custom JSON field name | `json:"user_id"` |
| `json:"-"` | Ignore field completely | `json:"-"` |
| `json:",omitempty"` | Omit if zero value | `json:"name,omitempty"` |
| `json:"name,omitempty"` | Custom name + omit if empty | `json:"email,omitempty"` |
| `json:",string"` | Encode number as string | `json:"id,string"` |

### **omitempty in Detail**

```go
type Profile struct {
    Name     string  `json:"name"`
    Age      int     `json:"age,omitempty"`      // Omit if 0
    Email    string  `json:"email,omitempty"`    // Omit if ""
    Active   bool    `json:"active,omitempty"`   // Omit if false
    Score    float64 `json:"score,omitempty"`    // Omit if 0.0
    Tags     []string `json:"tags,omitempty"`    // Omit if nil or empty
    Metadata map[string]string `json:"metadata,omitempty"` // Omit if nil
}

func main() {
    p := Profile{Name: "Alice"}
    
    data, _ := json.Marshal(p)
    fmt.Println(string(data))
    // {"name":"Alice"}
    // All zero-value fields are omitted!
}
```

### **Ignoring Fields**

```go
type User struct {
    ID       int    `json:"id"`
    Username string `json:"username"`
    Password string `json:"-"`           // Never included in JSON
    internal string                       // Unexported, also never included
}

func main() {
    user := User{
        ID:       1,
        Username: "alice",
        Password: "secret123",
        internal: "hidden",
    }
    
    data, _ := json.Marshal(user)
    fmt.Println(string(data))
    // {"id":1,"username":"alice"}
}
```

### **Handling Numbers as Strings**

```go
type Order struct {
    ID     int64   `json:"id,string"`      // Encode as "123" not 123
    Amount float64 `json:"amount,string"`  // Encode as "99.99" not 99.99
}

func main() {
    order := Order{ID: 12345, Amount: 99.99}
    
    data, _ := json.Marshal(order)
    fmt.Println(string(data))
    // {"id":"12345","amount":"99.99"}
    
    // Also works for unmarshaling
    jsonData := `{"id":"67890","amount":"123.45"}`
    var o Order
    json.Unmarshal([]byte(jsonData), &o)
    fmt.Printf("%+v\n", o)
    // {ID:67890 Amount:123.45}
}
```

### **Nested Structs**

```go
type Address struct {
    Street  string `json:"street"`
    City    string `json:"city"`
    Country string `json:"country"`
}

type Person struct {
    Name    string  `json:"name"`
    Address Address `json:"address"`  // Nested struct
}

func main() {
    p := Person{
        Name: "Bob",
        Address: Address{
            Street:  "123 Main St",
            City:    "NYC",
            Country: "USA",
        },
    }
    
    data, _ := json.MarshalIndent(p, "", "  ")
    fmt.Println(string(data))
    // {
    //   "name": "Bob",
    //   "address": {
    //     "street": "123 Main St",
    //     "city": "NYC",
    //     "country": "USA"
    //   }
    // }
}
```

### **Embedded Structs in JSON**

```go
type Timestamps struct {
    CreatedAt time.Time `json:"created_at"`
    UpdatedAt time.Time `json:"updated_at"`
}

type Article struct {
    ID    int    `json:"id"`
    Title string `json:"title"`
    Timestamps     // Embedded - fields are flattened
}

func main() {
    a := Article{
        ID:    1,
        Title: "Hello World",
        Timestamps: Timestamps{
            CreatedAt: time.Now(),
            UpdatedAt: time.Now(),
        },
    }
    
    data, _ := json.MarshalIndent(a, "", "  ")
    fmt.Println(string(data))
    // {
    //   "id": 1,
    //   "title": "Hello World",
    //   "created_at": "2024-01-15T10:30:00Z",
    //   "updated_at": "2024-01-15T10:30:00Z"
    // }
    // Note: Timestamps fields are at the top level, not nested!
}
```

### **Custom JSON Marshaling**

```go
type Status int

const (
    StatusPending Status = iota
    StatusActive
    StatusComplete
)

// Implement json.Marshaler interface
func (s Status) MarshalJSON() ([]byte, error) {
    var str string
    switch s {
    case StatusPending:
        str = "pending"
    case StatusActive:
        str = "active"
    case StatusComplete:
        str = "complete"
    default:
        str = "unknown"
    }
    return json.Marshal(str)
}

// Implement json.Unmarshaler interface
func (s *Status) UnmarshalJSON(data []byte) error {
    var str string
    if err := json.Unmarshal(data, &str); err != nil {
        return err
    }
    switch str {
    case "pending":
        *s = StatusPending
    case "active":
        *s = StatusActive
    case "complete":
        *s = StatusComplete
    default:
        return fmt.Errorf("unknown status: %s", str)
    }
    return nil
}
```

### **Reading Struct Tags via Reflection**

```go
import "reflect"

type User struct {
    Name  string `json:"name" validate:"required"`
    Email string `json:"email" validate:"email"`
}

func main() {
    t := reflect.TypeOf(User{})
    
    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        fmt.Printf("Field: %s\n", field.Name)
        fmt.Printf("  json tag: %s\n", field.Tag.Get("json"))
        fmt.Printf("  validate tag: %s\n", field.Tag.Get("validate"))
    }
    // Field: Name
    //   json tag: name
    //   validate tag: required
    // Field: Email
    //   json tag: email
    //   validate tag: email
}
```

### **Multiple Tags**

```go
type User struct {
    ID        int    `json:"id" db:"user_id" xml:"Id"`
    FirstName string `json:"first_name" db:"first_name" xml:"FirstName"`
    Email     string `json:"email" db:"email_address" xml:"Email" validate:"required,email"`
}
```

### **Common Validation Tags (with validator package)**

```go
import "github.com/go-playground/validator/v10"

type Registration struct {
    Username string `json:"username" validate:"required,min=3,max=20"`
    Email    string `json:"email" validate:"required,email"`
    Password string `json:"password" validate:"required,min=8"`
    Age      int    `json:"age" validate:"gte=18,lte=120"`
}

func main() {
    v := validator.New()
    
    reg := Registration{
        Username: "ab",  // Too short
        Email:    "invalid",
        Password: "short",
        Age:      15,
    }
    
    err := v.Struct(reg)
    if err != nil {
        for _, e := range err.(validator.ValidationErrors) {
            fmt.Printf("Field %s failed: %s\n", e.Field(), e.Tag())
        }
    }
}
```

### **Best Practices**

```go
// ✓ Good: Use snake_case for JSON (JavaScript convention)
type User struct {
    FirstName string `json:"first_name"`
    LastName  string `json:"last_name"`
}

// ✓ Good: Use omitempty for optional fields
type Request struct {
    Name     string  `json:"name"`
    Limit    int     `json:"limit,omitempty"`
    Offset   int     `json:"offset,omitempty"`
}

// ✓ Good: Use pointers for nullable fields
type Response struct {
    Data  *Result `json:"data,omitempty"`
    Error *string `json:"error,omitempty"`
}

// ✓ Good: Hide sensitive fields
type User struct {
    ID       int    `json:"id"`
    Password string `json:"-"`
}

// ✗ Avoid: Inconsistent naming
type Bad struct {
    firstName string `json:"FirstName"`  // Inconsistent case
}
```

## Interview Questions

**Q: What are struct tags in Go and what are they used for?**
**A:** Struct tags are string metadata attached to struct fields using backtick syntax. They provide instructions to packages that use reflection, such as `encoding/json` for JSON field names, `database/sql` for column mappings, and validation libraries for rules. Tags don't affect Go code directly but are read at runtime via the `reflect` package.

**Q: What does the `omitempty` JSON tag option do?**
**A:** `omitempty` tells the JSON encoder to skip fields that have zero values (0, "", false, nil, empty slices/maps). This produces cleaner JSON output by excluding fields that weren't set. Use it for optional fields to avoid cluttering JSON with empty values.

**Q: How do you completely exclude a field from JSON serialization?**
**A:** Use `json:"-"` as the tag. This completely ignores the field during both marshaling and unmarshaling. Alternatively, unexported fields (lowercase first letter) are automatically excluded since `encoding/json` can only access exported fields.

**Q: How do you handle embedded structs in JSON?**
**A:** Embedded (anonymous) structs have their fields flattened into the parent struct's JSON output by default. For nested output, use a named field instead. You can also add a JSON tag to the embedded field to change this behavior: `Timestamps Timestamps `json:"timestamps"`` creates nesting.
