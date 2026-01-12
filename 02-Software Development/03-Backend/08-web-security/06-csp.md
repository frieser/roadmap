---
---

## Summary
Content Security Policy (CSP) is an added layer of security that helps to detect and mitigate certain types of attacks, including Cross-Site Scripting (XSS) and data injection attacks.

## Detailed Explanation
CSP is implemented via an HTTP response header (`Content-Security-Policy`) that tells the browser which sources of content (scripts, styles, images) are trusted.

### Common Directives
- **`default-src 'self'`**: Only allow content from the same origin as the page.
- **`script-src`**: Define trusted sources for JavaScript.
- **`img-src`**: Define trusted sources for images.
- **`frame-ancestors 'none'`**: Prevent the site from being embedded in an iframe (protects against Clickjacking).

### Example Policy
`Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted-scripts.com;`
This policy allows scripts from the same domain and one specific external domain, but blocks everything else (like inline scripts or scripts from other domains).

## Go Context
You implement CSP by setting the header in your middleware.

### Example: Setting CSP in Go
```go
func cspMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.Header().Set("Content-Security-Policy", "default-src 'self'; script-src 'self' https://apis.google.com")
		next.ServeHTTP(w, r)
	})
}
```

## Interview Questions
- **Q: How does CSP prevent XSS?**
- **A:** XSS often involves an attacker injecting a malicious script tag into a page. CSP can block these injected scripts by only allowing scripts from a "whitelist" of trusted domains and by disabling the execution of inline scripts and `eval()`.

- **Q: What is `unsafe-inline` in CSP?**
- **A:** It is a keyword that allows the use of inline `<script>` tags and `onclick` attributes. Using `unsafe-inline` significantly weakens the protection provided by CSP and is generally discouraged.

- **Q: What is the `report-uri` directive?**
- **A:** It tells the browser where to send a JSON report if a CSP violation occurs. This allows developers to monitor and debug their policies in a real-world environment.
