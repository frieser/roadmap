#API
---
---

# Validate URL

## Summary

**Server-Side Request Forgery (SSRF)** is a critical vulnerability where an attacker tricks a backend server into making unauthorized requests to internal resources (e.g., cloud metadata services like `169.254.169.254`, internal databases, or administrative panels). **Open Redirects** occur when an application uses unvalidated user input to redirect a user to an external site, often used in phishing campaigns.

In Go, these are mitigated by strict URL parsing, scheme whitelisting, and low-level network validation to prevent "Time-of-Check to Time-of-Use" (TOCTOU) vulnerabilities like DNS Rebinding.

---

## Detailed Explanation

### 1. Robust URL Parsing
Always use the standard library's `net/url` package. Never attempt to parse URLs using regular expressions, as URL specifications are complex and prone to bypasses.

```go
u, err := url.Parse(userInput)
if err != nil {
    return fmt.Errorf("invalid URL")
}
```

### 2. Scheme Whitelisting
Attackers often use non-standard schemes like `file://`, `ftp://`, or `gopher://` to access local files or internal services.
*   **Best Practice**: Only allow `http` and `https`.

### 3. Blocking Internal IPs & Localhost (SSRF Prevention)
Validation must occur after DNS resolution. Validating the hostname string is insufficient because a domain like `malicious.com` could resolve to `127.0.0.1`.

*   **Loopback**: `127.0.0.0/8`, `::1`
*   **Private Ranges**: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`
*   **Link-Local/Metadata**: `169.254.169.254` (AWS/GCP/Azure metadata)

### 4. Preventing DNS Rebinding
To prevent DNS Rebinding, the IP address must be validated **at the moment the connection is established**. In Go, this is achieved by customizing the `DialContext` in `http.Transport`.

---

## Go Example: Secure SSRF Protection

This example demonstrates how to create a "safe" `http.Client` that blocks internal network ranges.

```go
package main

import (
	"context"
	"fmt"
	"net"
	"net/http"
	"net/url"
	"time"
)

// isPrivateIP checks if an IP belongs to a private, loopback, or link-local range.
func isPrivateIP(ip net.IP) bool {
	return ip.IsLoopback() || ip.IsLinkLocalUnicast() || ip.IsPrivate()
}

// GetSafeClient returns an http.Client that prevents SSRF by validating IPs during dialing.
func GetSafeClient() *http.Client {
	dialer := &net.Dialer{
		Timeout:   30 * time.Second,
		KeepAlive: 30 * time.Second,
	}

	transport := &http.Transport{
		DialContext: func(ctx context.Context, network, addr string) (net.Conn, error) {
			host, port, err := net.SplitHostPort(addr)
			if err != nil {
				return nil, err
			}

			// Resolve the IP addresses for the host
			ips, err := net.DefaultResolver.LookupIP(ctx, "ip", host)
			if err != nil {
				return nil, err
			}

			// Check if any of the resolved IPs are private/internal
			for _, ip := range ips {
				if isPrivateIP(ip) {
					return nil, fmt.Errorf("connection to internal IP %s is blocked", ip)
				}
			}

			// Use the first safe IP to dial (avoids TOCTOU)
			return dialer.DialContext(ctx, network, net.JoinHostPort(ips[0].String(), port))
		},
	}

	return &http.Client{
		Transport: transport,
		Timeout:   10 * time.Second,
		// Prevent Open Redirects by controlling redirect behavior
		CheckRedirect: func(req *http.Request, via []*http.Request) error {
			if len(via) >= 3 {
				return fmt.Errorf("too many redirects")
			}
			// Re-validate the new URL scheme and host if necessary
			return nil
		},
	}
}

func main() {
	client := GetSafeClient()
	
	// Example: Attempting to hit internal metadata
	_, err := client.Get("http://169.254.169.254/latest/meta-data/")
	if err != nil {
		fmt.Println("Blocked:", err) // Output: Blocked: connection to internal IP 169.254.169.254 is blocked
	}
}
```

---

## Interview Questions

### 1. What is the difference between a "Blind SSRF" and a "Full SSRF"?
In a **Full SSRF**, the attacker can see the response from the internal server (e.g., reading metadata). In a **Blind SSRF**, the attacker doesn't see the body but can infer information via timing or status codes (e.g., port scanning).

### 2. Why is it dangerous to validate a URL's hostname before making the request in Go?
Because of **DNS Rebinding**. An attacker can set up a DNS server that returns a safe IP (e.g., `1.1.1.1`) during your validation check, but returns a malicious IP (e.g., `127.0.0.1`) when the `http.Client` actually performs the dial.

### 3. How do you prevent Open Redirects when the redirect destination is a relative path?
Ensure the path starts with a single `/` and **not** `//`. A URL like `//google.com` is interpreted by browsers as a protocol-relative URL, which would redirect the user to an external site.

### 4. Which field in `http.Transport` is most critical for SSRF protection?
The `DialContext` (or `Dial`). It allows you to intercept the connection process after DNS resolution but before the TCP handshake, ensuring you are connecting to a validated IP address.

### 5. Does `url.Parse` validate if a URL is "safe"?
No. `url.Parse` only checks if the string follows the URI specification. It does not verify the scheme, the reputation of the domain, or whether the IP is internal. Validation logic must be implemented on top of the parsed object.
