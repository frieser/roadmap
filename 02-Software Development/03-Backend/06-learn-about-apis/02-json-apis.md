---
---

## Summary
JSON (JavaScript Object Notation) APIs are web services that use JSON as the primary format for data exchange. While often associated with REST, any API can return JSON. It has become the industry standard due to its simplicity, readability, and native support in modern programming languages.

## Detailed Explanation
JSON APIs succeeded XML-based APIs (like SOAP) because JSON is lighter and easier for browsers to parse.

### Key Characteristics
- **Key-Value Pairs**: Data is represented as keys and values (strings, numbers, booleans, arrays, or nested objects).
- **Text-Based**: Human-readable and easy to debug.
- **Language Independent**: Supported by virtually all modern programming languages.

### JSON API Specification (jsonapi.org)
There is a specific formal specification called "JSON:API" that provides conventions for:
- Resource objects.
- Relationships between resources.
- Sorting, filtering, and pagination.
- Error handling.
Using this specification reduces the number of "bikeshedding" decisions developers have to make about API structure.

## Go Context
Go provides excellent support for JSON through the `encoding/json` package.

### Example: Working with JSON in Go
```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

type Product struct {
	ID    int     `json:"id"`
	Name  string  `json:"name"`
	Price float64 `json:"price"`
}

func main() {
	// Marshaling (Go struct to JSON)
	p := Product{ID: 101, Name: "Laptop", Price: 1200.50}
	jsonData, _ := json.Marshal(p)
	fmt.Println(string(jsonData))

	// Unmarshaling (JSON to Go struct)
	jsonStr := `{"id":102, "name":"Mouse", "price":25.99}`
	var p2 Product
	err := json.Unmarshal([]byte(jsonStr), &p2)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Parsed: %+v\n", p2)
}
```

## Interview Questions
- **Q: Why is JSON preferred over XML for most modern APIs?**
- **A:** JSON is less verbose, resulting in smaller payloads. It is also much easier for JavaScript and other languages to parse into native objects without complex DOM manipulation.

- **Q: How do you handle custom field names in Go JSON tags?**
- **A:** You use struct tags like ``json:"my_custom_name"``. You can also use `,omitempty` to omit empty fields or `-` to ignore a field entirely.

- **Q: What is the JSON:API specification?**
- **A:** It is a standardized convention for building APIs in JSON. It defines how resources should be structured, how relationships should be linked, and how errors should be reported, ensuring consistency across different services.
