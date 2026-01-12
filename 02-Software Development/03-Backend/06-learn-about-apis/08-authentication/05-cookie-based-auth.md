---
---

## Summary
Cookie-based authentication is the traditional method for managing user sessions. The server stores session information in its memory or a database and sends a Session ID to the browser in a cookie. The browser then automatically includes this cookie in all future requests to the same domain.

## Detailed Explanation
This is often referred to as "Stateful Authentication" because the server must remember who is logged in.

### How it works
1. User submits login credentials.
2. Server verifies and creates a new session in the database/store.
3. Server sends a `Set-Cookie` header with the Session ID.
4. Browser stores the cookie.
5. On every request, the browser sends the cookie back.
6. Server looks up the Session ID to identify the user.

### Security Features
- **HttpOnly**: Prevents JavaScript from reading the cookie (protects against XSS).
- **Secure**: Ensures the cookie is only sent over HTTPS.
- **SameSite**: (Strict/Lax) Prevents the cookie from being sent in cross-site requests (protects against CSRF).

## Go Context
Go's `net/http` package makes it easy to set and read cookies. For session management, libraries like `gorilla/sessions` are popular.

### Example: Setting a Secure Cookie in Go
```go
func loginHandler(w http.ResponseWriter, r *http.Request) {
	// ... logic to verify user ...
	cookie := http.Cookie{
		Name:     "session_id",
		Value:    "random-unique-id",
		HttpOnly: true,
		Secure:   true,
		Path:     "/",
		MaxAge:   3600,
	}
	http.SetCookie(w, &cookie)
}
```

## Interview Questions
- **Q: What is CSRF and how does it relate to cookies?**
- **A:** Cross-Site Request Forgery is an attack where a malicious site tricks a logged-in user's browser into sending a request to your API. Since cookies are sent automatically by the browser, the request appears authenticated. This is mitigated by using CSRF tokens or the `SameSite` cookie attribute.

- **Q: What is the main drawback of cookie-based auth for mobile apps?**
- **A:** Mobile apps don't automatically manage cookies like browsers do. They usually prefer header-based tokens (JWT) which are easier to store and send manually.

- **Q: What happens if the session store (e.g., Redis) goes down?**
- **A:** Since the authentication is stateful, all users will be effectively logged out because the server can no longer verify their session IDs.
