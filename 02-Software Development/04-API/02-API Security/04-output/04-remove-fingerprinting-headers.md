#API
---
---

# Remove Fingerprinting Headers

## Summary
"Security through obscurity" is not a primary defense, but **Defense in Depth** mandates reducing the information available to an attacker. **Fingerprinting Headers** (like `Server`, `X-Powered-By`, `X-AspNet-Version`) reveal the specific technology stack and version numbers running on your server. This information allows attackers to target specific CVEs (Common Vulnerabilities and Exposures) known to affect those versions. Removing these headers increases the "reconnaissance cost" for an attacker.

## Detailed Explanation

### 1. Default Go Behavior
By default, the standard `net/http` library in Go does **not** send an `X-Powered-By` header. However, it does send a `Date` header (required by RFC) and sometimes a `User-Agent` if acting as a client.

Some third-party frameworks (like Gin or Fiber) might add their own headers or have default error pages that reveal the framework name.

### 2. Removing Headers in Go Frameworks

#### Standard Library
The standard library is clean by default. You only need to ensure you aren't manually adding them.

#### Gin
Gin doesn't add `X-Powered-By` by default, but it's good practice to ensure your middleware doesn't either.

#### Fiber
Fiber (inspired by Express.js) sends `X-Powered-By: Fiber` by default. You must disable it in the config.

```go
import "github.com/gofiber/fiber/v2"

func main() {
    app := fiber.New(fiber.Config{
        // Critical: Disable the fingerprinting header
        DisableStartupMessage: true,
        EnablePrintRoutes:     false,
        ServerHeader:          "", // Default is empty, but ensure it stays that way
    })
    
    // Fiber specific setting to remove "X-Powered-By: Fiber" isn't a direct config
    // but the header isn't sent if you don't enable it. 
    // However, some versions might send it via middleware.
}
```

**Note:** If you use a reverse proxy (Nginx) in front of Go, Nginx might add its own `Server: nginx/1.18.0`. You should configure Nginx (`server_tokens off;`) to hide the version.

### 3. Automated Scanners
Tools like **Wappalyzer**, **Shodan**, and automated botnets scan millions of IPs looking for banners like `Server: Apache/2.4.49`. If a vulnerability is found in 2.4.49, these bots instantly attack. By removing the version (or the header entirely), you drop off their easy-target list.

## Interview Questions

### 1. Is removing the `Server` header a complete security fix?
No. It is a hardening measure, not a fix. A determined attacker can still fingerprint your server by analyzing TCP timestamps, header ordering, or specific error page quirks. However, it stops low-effort automated scripts.

### 2. How do you remove the `Server` header in a standard Go `http.Server`?
Go's `net/http` does not add a `Server` header by default. If one is appearing, it's likely added by a middleware or a reverse proxy (like Nginx) sitting in front of your Go app.

### 3. Why might you want to keep the `Server` header but change its value?
Some HTTP clients or corporate proxies behave unpredictably if the `Server` header is missing entirely. A common practice is to set it to a generic value like `Server: API` or `Server: WebServer` to satisfy protocol expectations without revealing the technology stack.

### 4. What is the `X-Powered-By` header and who usually sets it?
It is a non-standard response header used to indicate the technology supporting the web application (e.g., `Express`, `ASP.NET`, `PHP/7.4`). It is almost always safe to remove and serves no functional purpose for the client.
