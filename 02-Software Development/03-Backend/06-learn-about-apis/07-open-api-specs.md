---
---

## Summary
The OpenAPI Specification (OAS), formerly known as Swagger, is a standard for defining and describing RESTful APIs. It allows humans and computers to discover and understand the capabilities of a service without access to source code.

## Detailed Explanation
An OpenAPI definition is a YAML or JSON file that describes an entire API.

### What it describes
- **Available endpoints** (`/users`, `/products`, etc.).
- **Operations** on each endpoint (`GET`, `POST`, etc.).
- **Input parameters** (query, path, headers).
- **Request bodies**.
- **Response codes and schemas**.
- **Authentication methods**.

### Ecosystem Benefits
- **Documentation**: Tools like Swagger UI generate interactive documentation directly from the spec.
- **Client Generation**: Tools like `openapi-generator` can create client SDKs in dozens of languages.
- **Server Stubs**: You can generate server boilerplates from the spec.
- **Mocking**: Servers like Prism can provide mock responses based on the spec for frontend development.

## Go Context
In Go, common tools include `swaggo/swag` (generates spec from code comments) or `deepmap/oapi-codegen` (generates code from a spec file).

### Example: Swag Comments in Go
```go
// @Summary Get user by ID
// @Description get string by ID
// @ID get-string-by-int
// @Accept  json
// @Produce  json
// @Param id path int true "User ID"
// @Success 200 {object} User
// @Router /users/{id} [get]
func GetUser(c *gin.Context) {
    // implementation
}
```

## Interview Questions
- **Q: What is the difference between Swagger and OpenAPI?**
- **A:** OpenAPI is the name of the specification standard (since it was donated to the Linux Foundation). Swagger is the set of commercial and open-source tools (Swagger UI, Swagger Editor) built by SmartBear around that specification.

- **Q: What are the advantages of Design-First vs. Code-First API development?**
- **A:** Design-First involves writing the OpenAPI spec before coding, which facilitates better collaboration, easier mocking, and consistent design. Code-First involves generating the spec from existing code, which is faster for small teams but can lead to "leaky abstractions" where the API reflects internal code structure too closely.

- **Q: How do you handle authentication in an OpenAPI spec?**
- **A:** You define a `securitySchemes` section in the components and then apply specific security requirements to endpoints. It supports API keys, Basic Auth, OAuth2, and OpenID Connect.
