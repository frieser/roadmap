---
---

# Apache HTTP Server

## 1. Summary
The Apache HTTP Server is a process-based web server that is highly modular. While often seen as "slower" than Nginx for high-concurrency tasks, it remains a robust choice for enterprise environments due to its flexibility and rich module ecosystem.

## 2. Detailed Explanation

### Multi-Processing Modules (MPMs)
Apache handles requests differently depending on the loaded MPM:
- **Prefork MPM**: Spawns a new process for each connection. Stable but consumes significant RAM.
- **Worker MPM**: Uses multiple processes, each with multiple threads. Better for scaling than Prefork.
- **Event MPM**: Similar to Nginx, it uses a listener thread to handle long-lived connections (Keep-Alive) efficiently.

### Key Features:
- **.htaccess**: Decentralized configuration at the directory level.
- **mod_rewrite**: Powerful URL manipulation engine.

## 3. Go-specific Context & Examples

### Using Apache as a Reverse Proxy
Apache uses `mod_proxy` to forward requests to backend Go services.

```apache
<VirtualHost *:80>
    ServerName my-go-app.com

    ProxyPreserveHost On
    ProxyPass / http://localhost:8080/
    ProxyPassReverse / http://localhost:8080/

    ErrorLog ${APACHE_LOG_DIR}/error.log
</VirtualHost>
```

### Comparison with Nginx (for Go)
| Feature | Nginx | Apache |
| :--- | :--- | :--- |
| **Model** | Event-driven (Asynchronous) | Process/Thread-based |
| **Memory** | Low footprint | Higher (process overhead) |
| **Ease of Proxy** | Very Simple | Simple (needs mod_proxy) |
| **Static Files** | Faster | Fast (but slightly more overhead) |

## 4. Interview Questions
1. **What is the purpose of `.htaccess`?**
   - To allow directory-level configuration without needing to restart the main server.
2. **Compare Apache's Prefork vs. Event MPM.**
   - Prefork is process-per-request (safe); Event is closer to Nginx's asynchronous model.
3. **Can Apache load balance Go instances?**
   - Yes, using `mod_proxy_balancer`.
