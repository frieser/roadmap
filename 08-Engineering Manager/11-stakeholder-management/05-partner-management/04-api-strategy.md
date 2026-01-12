---
---

## Summary
An API Strategy defines how a company exposes its functionality to the world (or internally). It treats the API as a **Product**, not just a technical pipe. Key decisions include Monetization, Versioning, Documentation, and Governance.

## Detailed Explanation

### Public vs Private vs Partner
1.  **Public**: Open to everyone. High documentation/support burden. Used for growth.
2.  **Partner**: Restricted access for specific business partners. Higher trust/SLA.
3.  **Private (Internal)**: Backend-for-Frontend (BFF) or microservices. Optimized for speed/agility.

### Developer Experience (DX)
*   **Documentation**: OpenAPI (Swagger) specs are mandatory.
*   **SDKs**: Provide client libraries in major languages (Go, Python, JS).
*   **Sandboxes**: Allow devs to test without a credit card.

### Lifecycle
*   **Versioning**: `/v1/users`. Don't break backward compatibility.
*   **Deprecation**: Clear "Sunset" policies (e.g., 6 months notice).

## Go-Specific Context/Examples

Go is the language of the cloud infrastructure, so many Infrastructure-as-Service (IaaS) APIs are built in Go.

### Designing a Go API Surface
*   **Idiomatic JSON**: Use `json:"user_id,omitempty"` tags.
*   **Context**: Always pass `context.Context` for cancellation/timeout.
*   **Errors**: Return structured errors, not plain strings.

## Interview Questions

**Q: API First vs Code First?**
**A:** **API First**: Design the schema (OpenAPI/Protobuf) *before* writing code. This allows Frontend and Backend to work in parallel and ensures the API is designed for the *consumer*, not the database schema.

**Q: How do you monetize an API?**
**A:**
1.  **Freemium**: Free up to X calls, then pay.
2.  **Tiered**: Basic endpoints free, Premium endpoints (analytics) paid.
3.  **Pay-as-you-go**: Metered billing ($0.001 per call).

**Q: REST vs GraphQL?**
**A:**
*   **REST**: Standard, cacheable (HTTP), good for server-to-server.
*   **GraphQL**: Flexible, single endpoint, solves over-fetching. Great for Frontends/Mobile apps with varying data needs.
