#API
---
---

# Safe Listed IPs (IP Whitelisting) in Go

## Summary
**IP Whitelisting** (or Safe Listing) is an access control mechanism that restricts API access to a predefined set of trusted IP addresses or CIDR ranges. It is commonly used for:
- **Private/Internal APIs**: Ensuring only internal microservices or VPNs can connect.
- **Admin Panels**: Restricting administrative access to office networks.
- **B2B Integrations**: Allowing only specific partner servers to hit webhooks.

## Detailed Explanation

### 1. The Challenge: Network Layers & Proxies
In a direct connection, `RemoteAddr` gives the client's IP. However, in modern cloud architectures, APIs usually sit behind Load Balancers (AWS ALB, Nginx, Cloudflare). In these cases, `RemoteAddr` is the Load Balancer's IP.

The client's real IP is passed in headers:
- **`X-Forwarded-For`**: The de-facto standard. A comma-separated list of IPs (`Client, Proxy1, Proxy2`).
- **`X-Real-IP`**: Used by some proxies (Nginx) to store the single client IP.

### 2. Security Risks: IP Spoofing
Trusting `X-Forwarded-For` blindly is dangerous. A malicious client can manually set `X-Forwarded-For: 127.0.0.1` to bypass checks.
**Solution**: Only trust the `X-Forwarded-For` header if the request comes from a **Trusted Proxy** (your own Load Balancer).

### 3. Go Implementation: CIDR Middleware
We typically validate IPs against a list of CIDR blocks (e.g., `192.168.1.0/24`).

```go
package main

import (
	"log"
	"net"
	"net/http"
	"strings"
)

// IPWhitelistMiddleware checks if the request comes from an allowed CIDR
func IPWhitelistMiddleware(allowedCIDRs []string, next http.Handler) http.Handler {
	// Parse CIDRs once at startup
	var nets []*net.IPNet
	for _, cidr := range allowedCIDRs {
		_, network, err := net.ParseCIDR(cidr)
		if err != nil {
			log.Fatalf("Invalid CIDR: %s", cidr)
		}
		nets = append(nets, network)
	}

	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		ipStr := GetRealIP(r)
		ip := net.ParseIP(ipStr)

		if ip == nil {
			http.Error(w, "Invalid IP", http.StatusForbidden)
			return
		}

		allowed := false
		for _, network := range nets {
			if network.Contains(ip) {
				allowed = true
				break
			}
		}

		if !allowed {
			log.Printf("Blocked IP: %s", ipStr)
			http.Error(w, "Forbidden: IP not allowed", http.StatusForbidden)
			return
		}

		next.ServeHTTP(w, r)
	})
}

// GetRealIP handles X-Forwarded-For logic
func GetRealIP(r *http.Request) string {
	// 1. Check X-Forwarded-For (Standard)
	xff := r.Header.Get("X-Forwarded-For")
	if xff != "" {
		// The client is the first IP in the list
		ips := strings.Split(xff, ",")
		return strings.TrimSpace(ips[0])
	}

	// 2. Check X-Real-IP (Nginx)
	xrip := r.Header.Get("X-Real-IP")
	if xrip != "" {
		return xrip
	}

	// 3. Fallback to RemoteAddr (Direct connection)
	ip, _, _ := net.SplitHostPort(r.RemoteAddr)
	return ip
}

func main() {
	// Allow only localhost and private network
	allowed := []string{"127.0.0.1/32", "10.0.0.0/8"}

	mux := http.NewServeMux()
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Welcome Trusted User!"))
	})

	log.Fatal(http.ListenAndServe(":8080", IPWhitelistMiddleware(allowed, mux)))
}
```

## Mermaid Flow: Request Filtering

```mermaid
graph TD
    Client[Client (1.2.3.4)] -->|Request| LB[Load Balancer]
    LB -->|X-Forwarded-For: 1.2.3.4| API[Go API Middleware]
    
    API -->|Parse IP| Check{Is IP in Allowed CIDRs?}
    Check -->|Yes| Handler[Business Logic]
    Check -->|No| Block[403 Forbidden]
    
    subgraph "Middleware Logic"
    Check
    end
```

## Interview Questions

1.  **Why is trusting `X-Forwarded-For` blindly considered a vulnerability?**
    *   *Answer:* Because the client controls the HTTP headers. An attacker can inject a fake `X-Forwarded-For` header. The application must only trust this header if the request originated from a trusted source (like the internal Load Balancer IP), effectively "sanitizing" the chain of trust.

2.  **What is a CIDR notation and how does Go handle it?**
    *   *Answer:* CIDR (Classless Inter-Domain Routing) represents a range of IPs (e.g., `10.0.0.0/24`). Go's `net.ParseCIDR` returns a `*net.IPNet` struct, which has a `.Contains(ip)` method to efficiently check if a specific IP falls within that range using bitwise operations.

3.  **How do you handle IPv4 vs IPv6 in whitelisting?**
    *   *Answer:* Go's `net.IP` type handles both IPv4 and IPv6 transparently. However, when defining your whitelist, you must explicitly include both IPv4 (e.g., `127.0.0.1/32`) and IPv6 (e.g., `::1/128`) ranges if you expect traffic from both protocols.

4.  **Can IP Whitelisting replace Authentication?**
    *   *Answer:* No. IP addresses can be spoofed (UDP), hijacked (BGP), or shared (NAT/CGNAT). Whitelisting should be used as a **Defense-in-Depth** layer *in addition to* strong authentication (mTLS, JWT, OAuth2), not as a replacement.
