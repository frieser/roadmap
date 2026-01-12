---
---

## Summary
CORS (Cross-Origin Resource Sharing) is a security feature implemented by browsers that allows or restricts web pages from making requests to a different domain than the one that served the web page.

## Detailed Explanation
Browsers enforce the **Same-Origin Policy** by default, which prevents a script on `site-a.com` from accessing data on `site-b.com`. CORS is the mechanism used to selectively relax this policy.

### How it works
1. The browser sends an `Origin` header with the request.
2. The server responds with an `Access-Control-Allow-Origin` header.
3. If the headers match, the browser allows the request; otherwise, it blocks it.

### Preflight Requests
For "unsafe" requests (like those with custom headers or `PUT`/`DELETE` methods), the browser first sends an **OPTIONS** request (the "preflight"). The server must approve this preflight before the browser sends the actual request.

## Go Context
In Go, you can implement CORS manually or use middleware like `rs/cors`.

### Example: Simple CORS Middleware in Go
```go
func corsMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.Header().Set("Access-Control-Allow-Origin", "https://trusted-site.com")
		w.Header().Set("Access-Control-Allow-Methods", "GET, POST, OPTIONS")
		
		if r.Method == "OPTIONS" {
			w.WriteHeader(http.StatusOK)
			return
		}
		next.ServeHTTP(w, r)
	})
}
```

## Interview Questions
- **Q: Is CORS a server-side security feature?**
- **A:** No, CORS is a **browser** security feature. It protects the **user**, not the server. An attacker can still make requests to your API using `curl` or a backend script, bypassing CORS entirely.

- **Q: What does `Access-Control-Allow-Origin: *` mean?**
- **A:** It allows any website in the world to make requests to your API and read the response. This is common for public APIs but dangerous for private APIs handling sensitive user data.

- **Q: Why does the preflight (OPTIONS) request exist?**
- **A:** It's a safety check for the server. It allows the server to tell the browser if a specific cross-origin request is allowed before the browser sends a potentially destructive request (like a `DELETE`).
