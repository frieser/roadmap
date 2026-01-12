#API
---
---

# HATEOAS (Hypermedia as the Engine of Application State)

## 1. Definition: The "Glory of REST"
HATEOAS is a constraint of the REST application architecture that represents the highest level of maturity in the **Richardson Maturity Model (Level 3)**.

- **Core Concept**: The client enters the API through a fixed entry point (the root URL) and discovers all subsequent actions and resources dynamically via hypermedia links provided in the server's responses.
- **Analogy**: Just like a human navigates a website by clicking links on a page without knowing the URL structure in advance, a HATEOAS-compliant client navigates an API using links returned by the server.

## 2. Common Formats
Standardized ways to represent links and resource relationships in JSON:

### **HAL (Hypertext Application Language)**
The most popular and simple format. It uses two reserved keys:
- `_links`: Contains links to related resources (e.g., `self`, `next`, `edit`).
- `_embedded`: Contains full resource objects related to the primary resource to reduce round-trips.

### **JSON:API**
A highly structured specification that aims to minimize both the number of requests and the amount of data transmitted. It uses a `links` object for navigation and a `relationships` object to describe connections between resources.

### **Siren**
A more complex format that describes both data and **actions**. It includes:
- **Entities**: The data.
- **Links**: Navigation.
- **Actions**: Methods (POST, PUT, DELETE) and fields required to transition state.

## 3. Pros and Cons

| **Pros** | **Cons** |
| :--- | :--- |
| **Discovery**: Clients can discover available actions dynamically. | **Complexity**: Increased complexity in both server and client implementation. |
| **Decoupling**: Servers can change URL structures without breaking clients (if they follow link relations). | **Payload Size**: Extra link data increases response size and bandwidth usage. |
| **Self-Documenting**: The response tells the client what it can do next based on the current state. | **Client Overhead**: Clients must be built to parse and follow links rather than hardcoding paths. |

## 4. Go: Implementation Patterns
In Go, HATEOAS is typically implemented using struct tags or response wrappers.

### **Option A: Struct Tags with Map**
Directly embedding links into the resource struct.

```go
type Link struct {
    Href string `json:"href"`
}

type UserResponse struct {
    ID    int             `json:"id"`
    Name  string          `json:"name"`
    Email string          `json:"email"`
    Links map[string]Link `json:"_links"`
}

// Usage
res := UserResponse{
    ID:   1,
    Name: "John Doe",
    Links: map[string]Link{
        "self":   {Href: "/users/1"},
        "update": {Href: "/users/1"},
        "delete": {Href: "/users/1"},
    },
}
```

### **Option B: Response Wrapper (Generic)**
A reusable wrapper to decorate any data with links.

```go
type HALResource struct {
    Data  any             `json:"data"`
    Links map[string]Link `json:"_links,omitempty"`
}

func Wrap(data any, self string) HALResource {
    return HALResource{
        Data: data,
        Links: map[string]Link{
            "self": {Href: self},
        },
    }
}
```

## 5. Interview Questions
1. **What is the Richardson Maturity Model, and what makes Level 3 different?**
   - *Answer*: Level 0 (Plain XML/JSON), Level 1 (Resources), Level 2 (HTTP Verbs), Level 3 (Hypermedia Controls/HATEOAS).
2. **How does HATEOAS improve API versioning?**
   - *Answer*: By encouraging clients to follow links instead of hardcoding URLs, servers can change endpoints or introduce new versions of resources without breaking compliant clients.
3. **What is the difference between `_links` and `_embedded` in HAL?**
   - *Answer*: `_links` contains navigation URLs; `_embedded` contains the actual resource data for related entities to save HTTP calls.
4. **Is HATEOAS always necessary?**
   - *Answer*: No. For simple internal APIs or mobile apps where the client is tightly controlled, the overhead often outweighs the benefits. It is most valuable for public, evolving ecosystems.
