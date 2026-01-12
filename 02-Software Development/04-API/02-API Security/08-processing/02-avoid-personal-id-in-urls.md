---
title: Avoid Personal ID in URLs
category: API Security
tags: [security, api, pii, privacy]
---

## Summary
Including Personal Identifiable Information (PII) or sensitive identifiers (like numeric user IDs, email addresses, or SSNs) in URLs is a significant security and privacy risk. URLs are frequently logged by web servers, proxies, and browsers, and can be leaked via the `Referer` header to third-party sites. This practice often leads to GDPR/CCPA compliance violations and increases the surface area for IDOR (Insecure Direct Object Reference) attacks.

## Detailed Explanation

### 1. The Risks
*   **Logging Leakage**: Standard web server configurations (Nginx, Apache, Cloud Access Logs) log the full request line. This means sensitive IDs are stored in plain text in access logs, which are often indexed by logging aggregators (ELK, Splunk) and accessible to unauthorized personnel.
*   **Browser History and Cache**: Browsers store URLs in the user's history and local cache. If a device is shared, these identifiers become visible to other users.
*   **Referer Header**: When a user navigates from a page with a sensitive URL to an external site, the browser sends the current URL in the `Referer` header. This leaks internal identifiers to third-party domains.
*   **Predictability (IDOR)**: Sequential numeric IDs in URLs (e.g., `/api/user/101`) allow attackers to "enumerate" resources by incrementing the ID, potentially accessing other users' data if authorization checks are weak.

### 2. Compliance Implications
*   **GDPR**: Under the General Data Protection Regulation, even pseudonymized identifiers that can be linked back to an individual are considered personal data. Leaking these in logs constitutes a data breach.
*   **CCPA**: The California Consumer Privacy Act treats internal identifiers as personal information, requiring strict controls over their disclosure.

### 3. Solutions and Best Practices
*   **Use Request Bodies**: For sensitive operations, move identifiers from the URL to the JSON body of a `POST`, `PUT`, or `PATCH` request.
*   **Opaque Identifiers (UUIDs/ULIDs)**: Replace sequential integers with non-deterministic identifiers like UUID v4. This prevents enumeration.
*   **Session-Based Context**: Instead of `/api/users/123/profile`, use `/api/me/profile`. The server should identify the user from the authenticated session (JWT or Session Cookie) rather than a URL parameter.
*   **Hashing/Tokenization**: If an ID must be in a URL, use a temporary, short-lived token or a cryptographically secure hash.

## Go (Golang) Application

### Bad Pattern: PII in Query Parameters
```go
// Bad: Email (PII) leaked in URL and logs
// GET /api/v1/user?email=john.doe@example.com
func GetUserByEmail(w http.ResponseWriter, r *http.Request) {
    email := r.URL.Query().Get("email")
    // ...
}
```

### Good Pattern: Moving sensitive ID to Body
```go
// Good: User context derived from authentication or passed in secure body
type UpdateProfileRequest struct {
    UserID string `json:"user_id"` // Using UUID
    Bio    string `json:"bio"`
}

func UpdateProfile(w http.ResponseWriter, r *http.Request) {
    var req UpdateProfileRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, "Invalid request", http.StatusBadRequest)
        return
    }
    // Perform authorization check: Does the session user match req.UserID?
}
```

### Good Pattern: Using UUIDs in Path
```go
// Good: Path uses an opaque UUID instead of a sequential ID
// GET /api/v1/users/550e8400-e29b-41d4-a716-446655440000
func GetUser(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    userID := vars["id"] // This is a UUID
    // ...
}
```

## Interview Questions

**Q: Why should you avoid putting an email address in a URL query parameter?**
**A:** Emails are PII. URLs are captured in web server access logs, proxy logs, and browser history. Additionally, if the page contains external links or resources (like images), the email address will be sent to those third parties via the `Referer` header.

**Q: How does using a UUID over a sequential integer improve security?**
**A:** UUIDs are non-deterministic and have a huge keyspace. This prevents "Resource Enumeration" or "BOLA" (Broken Object Level Authorization) attacks where an attacker guesses other resource IDs by simply incrementing a number.

**Q: What is the "Referer" header leak, and how does it affect API privacy?**
**A:** The `Referer` header contains the URL of the previous page. If an API is called by a frontend, and that frontend then navigates to a 3rd party site, the 3rd party site receives the full URL (including any sensitive IDs) of the frontend page in the HTTP request headers.

**Q: Explain the concept of "Session-Based Context" in API design.**
**A:** It is the practice of using endpoints like `/me` or `/my-account` where the specific User ID is not part of the URL. Instead, the server retrieves the User ID from the authentication token (e.g., a JWT claim) or a server-side session, ensuring the user can only ever reference their own context.
