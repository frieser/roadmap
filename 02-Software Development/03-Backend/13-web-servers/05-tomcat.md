---
---

# Apache Tomcat

## 1. Summary
Apache Tomcat is an open-source implementation of the **Jakarta Servlet**, Jakarta Expression Language, and WebSocket technologies. Unlike Nginx or Apache HTTPD, which are general-purpose web servers, Tomcat is a **Servlet Container** (or Application Server) specifically designed to run Java-based web applications.

## 2. Detailed Explanation
Tomcat handles the lifecycle of Java objects (Servlets) and provides the environment they need to execute.

### Tomcat vs. General Web Server
| Feature | Web Server (Nginx/Apache) | Tomcat (Servlet Container) |
| :--- | :--- | :--- |
| **Content** | Static + Request Routing | Dynamic Java execution |
| **Mapping** | URL to File/Proxy | URL to Java Class (Servlet) |
| **Logic** | Scripting/Modules | Full JVM capability |

### Architecture:
- **Connector**: Handles the HTTP communication (HTTP/1.1, HTTP/2, AJP).
- **Engine**: The top-level container that routes requests to virtual hosts.
- **Host**: Represents a network name (e.g., localhost).
- **Context**: Represents a single web application (usually a `.war` file).

## 3. Go-specific Context & Examples

### Coexistence with Go
In modern enterprise architectures, Go and Tomcat often coexist:
1. **Reverse Proxy (Shield)**: A Go-based API Gateway handles security/auth and proxies to legacy Tomcat apps.
2. **Sidecar**: A Go binary in the same pod manages logging, secrets, or service mesh for the Tomcat container.

### Replacing Tomcat (Strangler Fig Pattern)
Enterprises migrate from Java/Tomcat to Go for efficiency and simpler deployments:
- **Go**: Single static binary (~20MB), low RAM (~50MB).
- **Tomcat**: JVM + War (~200MB+), high RAM (~512MB+).

### Proxying to Tomcat in Go
```go
func main() {
    target, _ := url.Parse("http://tomcat-server:8080")
    proxy := httputil.NewSingleHostReverseProxy(target)
    
    http.ListenAndServe(":80", proxy)
}
```

## 4. Interview Questions
1. **Is Tomcat a Web Server or an Application Server?**
   - It is technically a Servlet Container, but often used as a standalone web server for Java apps.
2. **What is the purpose of the `web.xml` file in Tomcat?**
   - It is the deployment descriptor used to configure servlets and filters.
3. **Why would you replace a Tomcat app with a Go service?**
   - For better resource utilization, faster startup times, and simpler CI/CD pipelines.
