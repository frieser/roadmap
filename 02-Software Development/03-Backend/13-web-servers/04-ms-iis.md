---
---

# Microsoft IIS

## 1. Summary
Internet Information Services (IIS) is a flexible, secure, and manageable web server for hosting anything on the Microsoft Windows platform. It is deeply integrated with Windows Server and the .NET ecosystem.

## 2. Detailed Explanation
IIS uses a GUI-based management tool (IIS Manager) and is configured via XML (`web.config`).

### Architecture: Application Pools
- **Isolation**: IIS groups websites into **Application Pools**. Each pool runs in its own worker process (`w3wp.exe`).
- **Stability**: If one app crashes, other app pools remain unaffected.
- **Security**: Different pools can run under different service accounts.

## 3. Go-specific Context & Examples

### Running Go behind IIS (HttpPlatformHandler)
IIS manages the Go process lifecycle and passes a dynamic port via the `HTTP_PLATFORM_PORT` environment variable.

#### Go Code:
```go
func main() {
    port := os.Getenv("HTTP_PLATFORM_PORT")
    if port == "" {
        port = "8080" // Fallback for local dev
    }
    
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello from Go behind IIS!")
    })
    
    log.Fatal(http.ListenAndServe(":"+port, nil))
}
```

#### web.config:
```xml
<configuration>
  <system.webServer>
    <handlers>
      <add name="httpplatformhandler" path="*" verb="*" modules="httpPlatformHandler" resourceType="Unspecified" />
    </handlers>
    <httpPlatform processPath="c:\path\to\app.exe" arguments="">
    </httpPlatform>
  </system.webServer>
</configuration>
```

## 4. Interview Questions
1. **How does IIS pass the port to a Go application?**
   - Via the `HTTP_PLATFORM_PORT` environment variable.
2. **What are Application Pools?**
   - Isolated processes (`w3wp.exe`) that host web applications for better security and reliability.
3. **What module is required to run non-.NET apps in IIS?**
   - The **HttpPlatformHandler** or **FastCGI** module.
