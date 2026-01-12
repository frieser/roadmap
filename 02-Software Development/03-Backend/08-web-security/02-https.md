---
---

## Summary
HTTPS (Hypertext Transfer Protocol Secure) is an extension of HTTP. It uses TLS (Transport Layer Security) to encrypt the communication between a client and a server, ensuring data integrity, confidentiality, and authentication.

## Detailed Explanation
HTTPS is essentially HTTP sent over a secure, encrypted connection.

### The Three Pillars of HTTPS
1. **Encryption**: Hides the data from eavesdroppers. Even if an attacker intercepts the packets, they cannot read the content.
2. **Data Integrity**: Ensures that the data cannot be modified during transit without being detected.
3. **Authentication**: Proves that the user is communicating with the intended website (via SSL/TLS certificates).

### How it works
HTTPS uses a combination of asymmetric (public-key) and symmetric encryption.
- **Handshake**: Asymmetric encryption is used to securely exchange a "session key".
- **Data Transfer**: The session key (symmetric encryption) is used to encrypt the actual data being sent back and forth, as it is much faster than asymmetric encryption.

## Go Context
Go's `net/http` package makes it trivial to serve HTTPS.

### Example: Serving HTTPS in Go
```go
package main

import (
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Hello, Secure World!"))
	})

	// Requires cert.pem and key.pem
	log.Fatal(http.ListenAndServeTLS(":443", "cert.pem", "key.pem", nil))
}
```

## Interview Questions
- **Q: What is the difference between HTTP and HTTPS?**
- **A:** HTTP sends data in plain text, making it vulnerable to interception. HTTPS encrypts the data using SSL/TLS, providing security, privacy, and integrity.

- **Q: Can HTTPS protect against all types of attacks?**
- **A:** No. HTTPS protects data in transit. It does not protect against vulnerabilities in the application logic (like SQL injection), server-side breaches, or phishing.

- **Q: What happens if a certificate is expired?**
- **A:** The browser will show a warning to the user that the connection is not secure. From a technical standpoint, the encryption still works, but the *identity* of the server can no longer be trusted.
