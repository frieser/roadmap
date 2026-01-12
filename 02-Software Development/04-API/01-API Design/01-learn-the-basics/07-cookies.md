# Cookies

## Summary
HTTP cookies are small pieces of data sent from a server and stored on the user's browser. They are primarily used for state management (sessions, shopping carts), personalization (user preferences), and tracking. Since HTTP is a stateless protocol, cookies provide a way to maintain state between multiple requests from the same client.

## Detailed Explanation

### Attributes
Cookies have several attributes that control their behavior, security, and scope:

- **Secure**: When set, the cookie is only transmitted over encrypted connections (HTTPS). This prevents the cookie from being sent over cleartext HTTP, mitigating eavesdropping.
- **HttpOnly**: This attribute prevents client-side scripts (like JavaScript's `document.cookie`) from accessing the cookie. It is a critical defense against Cross-Site Scripting (XSS) attacks that attempt to steal session tokens.
- **SameSite**: Controls whether cookies are sent with cross-site requests. It helps mitigate Cross-Site Request Forgery (CSRF) attacks.
    - `Strict`: Cookie is only sent for first-party requests.
    - `Lax`: Default behavior. Cookie is sent for first-party requests and top-level navigations (like clicking a link).
    - `None`: Cookie is sent for all requests, including third-party (requires `Secure` attribute).
- **Domain**: Specifies which hosts are allowed to receive the cookie. If not specified, it defaults to the host that set the cookie (excluding subdomains).
- **Path**: Indicates a URL path that must exist in the requested URL for the browser to send the `Cookie` header.
- **Expires / Max-Age**: Determines the lifetime of the cookie. Without these, it's a session cookie (deleted when the browser closes).

### Security Implications
- **XSS (Cross-Site Scripting)**: If an attacker can inject JavaScript, they can read non-`HttpOnly` cookies. Using `HttpOnly` is the primary defense.
- **CSRF (Cross-Site Request Forgery)**: An attacker can trick a user's browser into sending a request to a site where the user is authenticated, using the browser's automatically attached cookies. `SameSite` attributes are the modern defense against this.
- **Session Hijacking**: Stealing a session cookie to impersonate a user. Using `Secure`, `HttpOnly`, and rotating session IDs helps prevent this.

## Go Application

In Go, the `net/http` package provides the `Cookie` struct and functions to manage them.

### http.Cookie Struct
```go
type Cookie struct {
    Name       string
    Value      string
    Path       string      // optional
    Domain     string      // optional
    Expires    time.Time   // optional
    RawExpires string      // for reading cookies only
    MaxAge     int
    Secure     bool
    HttpOnly   bool
    SameSite   SameSite
    Raw        string
    Unparsed   []string
}
```

### Setting a Cookie
```go
func setCookieHandler(w http.ResponseWriter, r *http.Request) {
    cookie := &http.Cookie{
        Name:     "session_token",
        Value:    "random-uuid-here",
        Path:     "/",
        HttpOnly: true,
        Secure:   true,
        SameSite: http.SameSiteStrictMode,
        MaxAge:   3600, // 1 hour in seconds
    }
    http.SetCookie(w, cookie)
    fmt.Fprintln(w, "Cookie has been set!")
}
```

### Reading a Cookie
```go
func getCookieHandler(w http.ResponseWriter, r *http.Request) {
    cookie, err := r.Cookie("session_token")
    if err != nil {
        if err == http.ErrNoCookie {
            http.Error(w, "No cookie found", http.StatusUnauthorized)
            return
        }
        http.Error(w, "Internal error", http.StatusInternalServerError)
        return
    }
    fmt.Fprintf(w, "Found cookie: %s = %s", cookie.Name, cookie.Value)
}
```

### Deleting a Cookie
To delete a cookie, you set it with a `MaxAge` of -1 or an `Expires` date in the past.
```go
func deleteCookieHandler(w http.ResponseWriter, r *http.Request) {
    cookie := &http.Cookie{
        Name:   "session_token",
        Value:  "",
        Path:   "/",
        MaxAge: -1,
    }
    http.SetCookie(w, cookie)
}
```

## Interview Questions

**Q: What is the difference between HttpOnly and Secure cookie attributes?**
**A:** `HttpOnly` prevents client-side JavaScript from accessing the cookie, protecting against XSS-based token theft. `Secure` ensures the cookie is only sent over HTTPS, protecting against interception over unencrypted connections.

**Q: How does the SameSite attribute help prevent CSRF?**
**A:** `SameSite=Strict` or `Lax` prevents the browser from sending the cookie with cross-site requests (e.g., when a user is on `attacker.com` and a request is made to `bank.com`). Since the session cookie isn't sent, the cross-site request is not authenticated.

**Q: What happens if you don't set an Expires or Max-Age attribute on a cookie?**
**A:** It becomes a "Session Cookie," which is stored in temporary memory and is typically deleted when the user closes the browser or ends the session.

**Q: Can a subdomain access a cookie set by its parent domain?**
**A:** Yes, if the parent domain sets the `Domain` attribute (e.g., `Domain=example.com`), subdomains like `api.example.com` will receive the cookie. If the `Domain` attribute is omitted, it defaults to the host only, excluding subdomains.
