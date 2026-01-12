---
---

# IIS (Internet Information Services)

IIS is an extensible web server created by Microsoft for use with the Windows NT family. For DevOps engineers working in Windows-heavy environments (Azure, .NET shops), mastering IIS is essential.

## Summary

IIS is a feature-rich web server that supports HTTP, HTTPS, FTP, FTPS, SMTP, and NNTP. It is tightly integrated with the Windows OS. Its core strength lies in hosting **ASP.NET** (Core and Framework) applications via the **ASP.NET Core Module (ANCM)**, but it can also host PHP, Python, and Go via FastCGI or HttpPlatformHandler.

## Detailed Explanation

### 1. Key Concepts
*   **Application Pools**: The isolation boundary. Each App Pool runs in its own worker process (`w3wp.exe`). If one app crashes (e.g., memory leak), it doesn't affect apps in other pools.
*   **ISAPI Filters**: DLLs that filter requests (legacy).
*   **HttpPlatformHandler**: A module that allows IIS to launch and manage any executable (like `java.exe`, `python.exe`, or a `go-binary.exe`) and proxy traffic to it.

### 2. Integration with Windows
*   **Windows Authentication**: Seamless Single Sign-On (SSO) for corporate intranets using Active Directory (Kerberos/NTLM).
*   **Performance Monitor**: Deep integration with Windows PerfMon for tracking request queues, CPU, and memory.

---

## Go Implementation Example

To run a Go application behind IIS, you don't use FastCGI. The modern approach is using the **HttpPlatformHandler** or simply configuring IIS as a **Reverse Proxy** (using URL Rewrite + Application Request Routing).

### IIS Configuration (web.config)
This XML file tells IIS to launch your Go binary and forward traffic to it.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <handlers>
            <add name="GoHandler" path="*" verb="*" modules="HttpPlatformHandler" resourceType="Unspecified" />
        </handlers>
        <!-- 
            processPath: The path to your Go binary
            arguments: Any flags your app needs
            HTTP_PLATFORM_PORT: IIS sets this env var randomly. Your Go app must listen on it.
        -->
        <httpPlatform processPath="C:\inetpub\wwwroot\myapp.exe"
                      startupTimeLimit="60"
                      stdoutLogEnabled="true"
                      stdoutLogFile=".\logs\go-app.log">
            <environmentVariables>
                <environmentVariable name="GO_ENV" value="production" />
            </environmentVariables>
        </httpPlatform>
    </system.webServer>
</configuration>
```

### Go Application
The Go app must listen on the port provided by IIS in the environment variable.

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"os"
)

func main() {
	// IIS HttpPlatformHandler sets this variable
	port := os.Getenv("HTTP_PLATFORM_PORT")
	if port == "" {
		port = "8080" // Fallback for local dev
	}

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Hello from Go running inside IIS on port %s!", port)
	})

	fmt.Printf("Listening on port %s...\n", port)
	log.Fatal(http.ListenAndServe(":"+port, nil))
}
```

## Interview Questions

**Q: What is an Application Pool in IIS and why is it important?**
**A:** An Application Pool serves as an isolation boundary. It allows you to group one or more websites into a dedicated worker process (`w3wp.exe`). If a website in "AppPool A" crashes or consumes 100% CPU, websites in "AppPool B" remain unaffected. It also allows you to run different apps with different .NET versions or security identities.

**Q: How do you host a non-.NET application (like Go or Node.js) in IIS?**
**A:** You use the **HttpPlatformHandler** module (or the newer ASP.NET Core Module for Node). This module allows IIS to act as a process manager: it starts the external executable (`node.exe`, `myapp.exe`), injects a dynamic port via an environment variable, and proxies incoming HTTP traffic to that process.

**Q: What is "Application Request Routing" (ARR)?**
**A:** ARR is an extension that turns IIS into a powerful Reverse Proxy and Load Balancer. It allows IIS to forward traffic to other servers (or local processes) based on rules defined in the URL Rewrite module. It is the IIS equivalent of Nginx's `proxy_pass`.
