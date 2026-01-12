#API
---
---

## Summary

Content Negotiation is the mechanism that allows a client and a server to agree on the best representation for a given resource. This process ensures that the response is delivered in a format (JSON, XML), language (English, Spanish), or encoding (Gzip, Brotli) that the client can understand and process efficiently. It is a fundamental part of the HTTP protocol that enables building truly RESTful and flexible APIs.

## Detailed Explanation

### Server-driven vs Client-driven Negotiation

1.  **Server-driven (Proactive) Negotiation**: 
    The server selects the best representation based on the information provided by the client in HTTP headers.
    -   **How it works**: The client sends headers like `Accept` or `Accept-Language`. The server examines these, compares them with available representations, and returns the best match.
    -   **Pros**: Efficient (single round-trip), simple for clients.
    -   **Cons**: The server must "guess" the client's preference if headers are ambiguous; caching becomes complex (requires the `Vary` header).

2.  **Client-driven (Reactive) Negotiation**: 
    The server provides a list of available representations, and the client chooses the one it wants.
    -   **How it works**: The server responds with a `300 Multiple Choices` or `406 Not Acceptable` status code and a list of links to different versions.
    -   **Pros**: Most accurate selection; offloads decision-making to the client.
    -   **Cons**: Requires an extra round-trip; more complex for the client to implement.

### Key Negotiation Headers

*   **`Accept`**: Specifies the media types (MIME types) that are acceptable for the response.
    *   Example: `Accept: application/json, text/plain;q=0.9`
*   **`Accept-Language`**: Specifies the preferred natural languages.
    *   Example: `Accept-Language: en-US, en;q=0.8, es;q=0.5`
*   **`Accept-Encoding`**: Specifies the compression algorithms the client supports.
    *   Example: `Accept-Encoding: gzip, deflate, br`
*   **`Content-Type`**: While not for negotiation itself, it indicates the media type of the resource being sent in the body (request or response).

### Quality Values (q-factor)

HTTP uses **Quality Values** (or q-factors) to weight preferences. They range from `0.0` to `1.0`.
-   A higher value indicates a higher preference.
-   If no `q` is specified, it defaults to `1.0`.
-   Example: `Accept: application/json;q=1.0, application/xml;q=0.8` means the client prefers JSON but will accept XML if JSON is unavailable.

## Go Implementation

In Go, content negotiation is often handled in middleware or within specific handlers using the standard `net/http` package and specialized libraries for complex cases like internationalization.

### 1. Simple Format Negotiation
You can manually check the `Accept` header to decide which formatter to use.

```go
func handler(w http.ResponseWriter, r *http.Request) {
    accept := r.Header.Get("Accept")
    
    if strings.Contains(accept, "application/json") {
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(data)
        return
    }
    
    // Fallback to XML or plain text
    w.Header().Set("Content-Type", "application/xml")
    xml.NewEncoder(w).Encode(data)
}
```

### 2. Language Negotiation with `golang.org/x/text`
For robust language negotiation, the `golang.org/x/text/language` package is the standard tool.

```go
package main

import (
    "fmt"
    "net/http"
    "golang.org/x/text/language"
)

var supportedLanguages = []language.Tag{
    language.English, // First one is the fallback
    language.Spanish,
    language.French,
}

var matcher = language.NewMatcher(supportedLanguages)

func i18nHandler(w http.ResponseWriter, r *http.Request) {
    // Parse the Accept-Language header
    t, _, _ := language.ParseAcceptLanguage(r.Header.Get("Accept-Language"))
    
    // Find the best match among supported languages
    tag, _, _ := matcher.Match(t...)
    
    fmt.Fprintf(w, "Best match for your preference: %s\n", tag.String())
}
```

## Interview Questions

**Q: What is the difference between `Accept` and `Content-Type`?**
**A:** `Accept` is a request header used by the client to tell the server what formats it can handle. `Content-Type` is used in both requests and responses to indicate the actual media type of the entity-body being sent.

**Q: Why is the `Vary` header important in Content Negotiation?**
**A:** The `Vary` header tells downstream caches (like CDNs or browsers) that the response depends on specific request headers (e.g., `Vary: Accept-Language`). This prevents a user requesting Spanish content from receiving a cached English version.

**Q: What happens if the server cannot satisfy any of the client's `Accept` preferences?**
**A:** The server should ideally return a `406 Not Acceptable` status code, although many servers simply fallback to a default representation (like JSON).

**Q: Explain how q-factors work.**
**A:** Q-factors (Quality Values) are decimal numbers from 0 to 1 that indicate the relative priority of items in a comma-separated list. A higher value means higher priority. For example, `text/html;q=0.9, application/xhtml+xml;q=1.0` prioritizes XHTML over HTML.
