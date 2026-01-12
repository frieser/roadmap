# Simple JSON APIs

## Summary
Simple JSON APIs (often called "Pragmatic REST") are the most common style for modern web services. Unlike strict specifications like JSON:API or HATEOAS, they focus on ease of consumption and human readability by using standard HTTP methods and status codes with a flat or slightly enveloped JSON structure.

## Detailed Explanation

### What makes an API "Simple JSON"?
Simple JSON APIs follow a pragmatic approach to REST. They use the core principles (Resource-based URLs, HTTP methods) but ignore more complex requirements like hypermedia links (HATEOAS).
- **Pragmatism over Purity**: Focus on what's easy for frontend developers to consume.
- **Resource Orientation**: URLs like \`/users/123\` represent resources.
- **HTTP Semantics**: Use \`GET\` for retrieval, \`POST\` for creation, \`PUT\`/\`PATCH\` for updates, and \`DELETE\` for removal.
- **Statelessness**: Each request contains all information needed to process it.

### JSON Structure: Envelopes vs Raw Objects
There are two main patterns for structuring the response body:

#### 1. Raw Objects (Flat)
The response body is the resource itself.
\`\`\`json
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com"
}
\`\`\`
- **Pros**: Minimal overhead, matches client-side models directly.
- **Cons**: Difficult to add metadata (pagination, warnings) without breaking changes.

#### 2. Envelopes
The resource is wrapped in a "data" property, allowing for top-level metadata.
\`\`\`json
{
  "data": {
    "id": 1,
    "name": "John Doe"
  },
  "meta": {
    "version": "1.0",
    "request_id": "req_abc123"
  }
}
\`\`\`
- **Pros**: Consistent structure, easy to add extra context (pagination total, execution time).
- **Cons**: Slightly more verbose, extra nesting.

### Content-Type
Simple JSON APIs MUST use the standard \`application/json\` Content-Type. This ensures that browsers, CLI tools (like \`curl\`), and all modern programming languages can automatically parse the body.

### Go: Implementation with \`encoding/json\`
In Go, the standard library \`encoding/json\` is the tool of choice.

#### Struct Tags and \`omitempty\`
Struct tags allow you to map Go struct fields to JSON keys. The \`omitempty\` option is crucial for keeping payloads clean.

\`\`\`go
type User struct {
    ID    int    \`json:"id"\`
    Name  string \`json:"name"\`
    Email string \`json:"email,omitempty"\` // Omitted if empty string
    Bio   *string \`json:"bio,omitempty"\`  // Omitted if nil
}
\`\`\`

#### Handling Nulls vs Omitted Fields
- **Pointers**: Using \`*string\` allows you to distinguish between an empty string (\`""\`) and a null value (\`nil\`). If the field is \`nil\`, \`omitempty\` will remove it from the output.
- **Null values**: If a field must explicitly be \`null\` in JSON, don't use \`omitempty\` on a pointer field.

\`\`\`go
// Example of handling nullable fields in Go
func main() {
    bio := "Software Engineer"
    u := User{
        ID:   1,
        Name: "Alice",
        Bio:  &bio,
    }
    
    data, _ := json.Marshal(u)
    fmt.Println(string(data))
}
\`\`\`

## Interview Questions

**Q: What is the difference between \`application/json\` and \`text/javascript\`?**
**A:** \`application/json\` is the official MIME type for JSON data. \`text/javascript\` (or \`application/javascript\`) is used for executable scripts. While JSON is valid JS syntax, the \`application/json\` type provides better security and tells the client to treat the content as data, not code.

**Q: How do you handle a zero value (like \`0\` or \`false\`) that should be included in JSON even if \`omitempty\` is set?**
**A:** \`omitempty\` treats \`0\`, \`false\`, and \`""\` as empty. If you need to send these values but omit the field if it's truly "missing", use a pointer (e.g., \`*int\`, \`*bool\`). A \`nil\` pointer will be omitted, but a pointer to \`0\` will be serialized as \`0\`.

**Q: When should you prefer an envelope over a raw object?**
**A:** Prefer envelopes when you need to return metadata alongside the primary data (e.g., pagination info like \`total_pages\`, or \`warnings\`). If the API is purely internal and performance/simplicity is paramount, raw objects are often easier to manage.

**Q: Why is JSON:API (the spec) different from a "Simple JSON API"?**
**A:** JSON:API is a strict, opinionated specification that mandates a specific structure (\`data\`, \`attributes\`, \`relationships\`) and includes HATEOAS features. A "Simple JSON API" is more flexible and doesn't follow a formal multi-industry spec, focusing instead on pragmatic REST principles.
