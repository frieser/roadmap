# API
---
---

# HTTP Status Codes

HTTP Status Codes are standardized three-digit integers returned by a server in response to a client's request. They indicate whether a request was successful, failed, or requires further action.

## 1. Status Code Categories
The first digit of the status code defines the class of response:

| Class | Category | Description |
| :--- | :--- | :--- |
| **1xx** | Informational | Request received, continuing process. |
| **2xx** | Success | Action successfully received, understood, and accepted. |
| **3xx** | Redirection | Further action must be taken to complete the request. |
| **4xx** | Client Error | Request contains bad syntax or cannot be fulfilled. |
| **5xx** | Server Error | Server failed to fulfill an apparently valid request. |

## 2. Common HTTP Status Codes

### **Successful (2xx)**
*   **200 OK**: The request was successful. The meaning of "success" depends on the method (GET: resource fetched, POST: result transmitted).
*   **201 Created**: The request succeeded, and a new resource was created (common for POST/PUT).
*   **204 No Content**: The request succeeded, but there is no content to send in the response body (common for DELETE).

### **Client Errors (4xx)**
*   **400 Bad Request**: The server cannot process the request due to client error (e.g., malformed syntax, invalid data).
*   **401 Unauthorized**: The client must authenticate itself to get the requested response. (Semantically "Unauthenticated").
*   **403 Forbidden**: The client is authenticated but does not have permission to access the resource.
*   **404 Not Found**: The server cannot find the requested resource (endpoint or specific data).

### **Server Errors (5xx)**
*   **500 Internal Server Error**: A generic error message when the server encounters an unexpected condition.
*   **502 Bad Gateway**: The server, acting as a gateway or proxy, received an invalid response from the upstream server.

## 3. Best Practices for API Responses
1.  **Use the Correct Code**: Don't return `200 OK` with an error message in the body.
2.  **Be Consistent**: Use the same status codes for the same types of outcomes across your entire API.
3.  **Provide Context**: For error codes (4xx/5xx), return a JSON body explaining *why* the error occurred (e.g., following [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807)).
4.  **Idempotency**: Ensure that retrying a request with certain status codes (like 5xx) doesn't cause side effects if the first one actually succeeded partially.
5.  **Rate Limiting**: Use **429 Too Many Requests** when a client exceeds their quota, ideally with a `Retry-After` header.

## 4. Setting Status Codes in Go
In Go's `net/http` package, you set the status code using the `WriteHeader` method of the `http.ResponseWriter`.

**Crucial**: You must call `WriteHeader` **before** writing any data to the response body (e.g., via `w.Write` or `json.NewEncoder(w).Encode`). If you don't call it, the first call to `Write` will automatically send a `200 OK`.

```go
func handler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        // Set status code to 405 Method Not Allowed
        w.WriteHeader(http.StatusMethodNotAllowed)
        return
    }

    // Set headers before writing the status and body
    w.Header().Set("Content-Type", "application/json")
    
    // Set status code to 201 Created
    w.WriteHeader(http.StatusCreated)
    
    // Write response body
    json.NewEncoder(w).Encode(map[string]string{"message": "Resource created"})
}
```

## 5. Interview Questions
1.  **What is the difference between 401 Unauthorized and 403 Forbidden?**
    *   *Answer*: 401 means the user is not authenticated (identity unknown); 403 means the user is authenticated but lacks permission (identity known, but action denied).
2.  **When should you use a 204 No Content response?**
    *   *Answer*: Usually for DELETE requests or PUT updates where the client doesn't need to see the updated resource, saving bandwidth by omitting the body.
3.  **What does a 502 Bad Gateway error typically indicate in a microservices architecture?**
    *   *Answer*: It means a proxy (like Nginx or an API Gateway) received an invalid response from a backend service it was trying to reach.
4.  **Why should you avoid returning 200 OK for every request and putting the error in the body?**
    *   *Answer*: It breaks standard HTTP tooling (proxies, caches, monitoring) that relies on status codes to determine success/failure and makes the API harder to consume correctly.

---
**Sources**:
*   [MDN Web Docs: HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status)
*   [Roadmap.sh: API Design](https://roadmap.sh/api-design)
