---
tags: ['docker', 'containers', 'debugging', 'development', 'devops', 'tools', 'roadmap']
---

# Debuggers for Containers

## Summary

Debugging containerized applications requires specialized tools since you can't attach traditional debuggers to running containers. Popular approaches include remote debugging (exposing debug ports), language-specific debuggers (Delve for Go, pdb for Python), IDE integration (VS Code, IntelliJ), and logging strategies for production troubleshooting.

## Detailed Explanation

### Remote Debugging with Delve (Go)

```bash
# DELVE - Go Debugger with Remote Support
# Delve runs in container, connects from host

# BUILD DEBUG IMAGE
# From earlier example - includes Delve
FROM golang:1.21-alpine
# ... Delve installed ...

# RUN WITH DEBUGGER LISTENING
docker run -d \
  --name go-app \
  -p 2345:2345 \
  -p 4000:4000 \
  myapp-with-delve:latest

# CONNECT FROM HOST
dlv connect :2345
# Connects Delve on host to container's debug port
# Breakpoints will be set from host
# Container process runs under debugger control

# SET BREAKPOINTS
dlv break main.go:45
(dlv) break main.go:78

# CONTINUE EXECUTION
(dlv) continue

# STEP THROUGH CODE
(dlv) next
(dlv) step

# INSPECT VARIABLES
(dlv) locals
(dlv) print main.Name

# DISCONNECT
(dlv) disconnect
```

```yaml
# DOCKER-COMPOSE.YML FOR GO DEBUGGING
version: '3.8'

services:
  go-app:
    build: .
    ports:
      - "2345:2345"  # Delve port
      - "4000:4000"  # Application port
    command: ["/usr/local/bin/delve", "--listen=:2345", "--headless", "--api-version=2", "exec", "/app/app"]

  debugger:
    build: .
    command: ["/usr/local/bin/delve", "connect", ":2345"]
    depends_on:
      - go-app
```

### Remote Debugging with Python (pdb)

```bash
# PDB - Python Debugger
# Standard Python debugger, requires TCP communication

# RUN PYTHON WITH PDB SERVER
docker run -d \
  --name python-app \
  -p 8000:8000 \
  python -m pdb -m pdb.Pdb(addr=('0.0.0.0', 8000)) myapp.py

# CONNECT FROM HOST
telnet localhost 8000
# At (Pdb) prompt, enter: c to continue execution

# SET BREAKPOINTS
b main.py:10  # At (Pdb) prompt
(b) myfunction  # Continue execution

# DEBUGGING COMMANDS
(b) next      # Step through code
(b) step      # Step to next line
(b) continue    # Continue to next breakpoint

# INSPECT VARIABLES
(b) locals
(b) p my_var
```

```yaml
# DOCKER-COMPOSE WITH PDB
version: '3.8'

services:
  python-app:
    build: .
    command: python -m pdb -m pdb.Pdb(addr=('0.0.0.0', 8000)) myapp.py
    ports:
      - "8000:8000"
    stdin_open: true  # Enable interactive PDB
    tty: true
```

### Remote Debugging with Node.js

```bash
# NODE.JS INSPECTOR
# Chrome DevTools Protocol
# Attach to container's remote inspector

# RUN WITH INSPECTOR ENABLED
docker run -d \
  --name node-app \
  -p 3000:9229 \
  --inspect=node \
  myapp:latest

# CONNECT FROM HOST
# Open Chrome DevTools
# chrome-devtools://localhost:9229
# View Sources, Console, Network

# USE --INSPECT FLAG (V8+)
docker run -d \
  --inspect=node:latest \
  myapp:latest

# VS CODE INTEGRATION
# VS Code connects to inspector
# Set breakpoints and debug as usual

# ALTERNATIVE: ndb
docker run -d \
  -p 9229:9229 \
  myapp:latest
  --command "node --inspect=0.0.0.0 node_modules/.bin/ndb"
```

### IDE Integration

```yaml
# VS CODE DEV CONTAINERS
# VS Code Remote Development Extension
# Attach to running containers

# docker-compose.yml for VS Code
version: '3.8'

services:
  app:
    build: .
    volumes:
      - .:/workspace
      - /root/.vscode-server
    command: sleep infinity
    # VS Code can attach to container via Remote SSH

# INTELLIJ IDEA
# Docker Compose plugin
# Connect to containers directly from IDE

# JETBRAINS GATEWAY
# Connect via SSH to development containers
```

### Logging for Debugging

```yaml
# LOGGING CONFIGURATION
version: '3.8'

services:
  app:
    image: myapp:latest
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    environment:
      - LOG_LEVEL=debug
      - LOG_FORMAT=json
```

```bash
# VIEW APPLICATION LOGS
docker logs -f app
docker logs --tail 100 app

# LOGS WITH CONTEXT
docker logs --since 2024-01-01T10:00:00 app
docker logs --until 2024-01-01T11:00:00 app

# LOGS TO FILE
docker logs app > app.log

# STRUCTURED LOGS
docker run -d \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=5 \
  myapp:latest
```

### Production Debugging

```yaml
# PRODUCTION DEBUGGING PRACTICES

# 1. LOGGING, NOT DEBUGGING
# Enable comprehensive logging
# Use structured log formats (JSON)
# Log to external systems (ELK, Splunk)

# 2. DISTRIBUTED TRACING
# Add tracing (OpenTelemetry, Jaeger)
# Debug across microservices

# 3. HEALTH CHECKS
# Implement /healthz endpoint
# Use readiness/liveness probes

# 4. GRACEFUL SHUTDOWN
# Handle SIGTERM properly
# Save state before shutdown

# 5. DEBUGGING FLAGS
# Enable debug mode for emergencies only
# Never in production builds

# 6. OBSERVABILITY
# Metrics, dashboards
# Prometheus + Grafana
# Application Performance Monitoring

# DEBUGGING IN KUBERNETES
# kubectl describe pod mypod
# kubectl logs mypod
# Use sidecar containers with debug tools
# kubectl exec -it mypod -- /bin/sh
```

### Security Considerations

```yaml
# SECURITY FOR DEBUGGING

# Never expose debug ports publicly
# - Use SSH tunnels instead
# - Restrict access to VPN

# Debug containers should have resource limits
# Set memory and CPU limits
# Prevent debugger from consuming all resources

# Clean up debug containers
# Remove when not in use
# Don't leave debug environments running

# Use separate debug images
# Debug images with extra tools
# Production images should be minimal and secure

# Enable audit logging
# Log all debugging actions
# Track who accessed debug interfaces
```

### Common Debugging Scenarios

```yaml
# DEADLOCK DEBUGGING
# Check for stuck processes
# Review logs for lock contention
# Use thread dump tools (Go: pprof)

# MEMORY LEAKS
# Monitor container memory usage
# Use profiling tools
# Check for goroutine leaks (Go)

# PERFORMANCE ISSUES
# Profile application in production mode
# Use sampling profilers
# Check CPU and I/O bottlenecks

# CRASH INVESTIGATION
# Get core dumps
# docker exec myapp gcore <pid>
# Analyze with debugger

# NETWORK ISSUES
# Test connectivity between services
# Check firewall rules
# Verify DNS resolution

# CONCURRENCY BUGS
# Use race detector (Go: go run -race)
# Add mutex logging
# Test under load
```

## Interview Questions

### Q1: How do you debug a Go application running in Docker?
**A:** Use Delve (Go debugger) by exposing debug port (`-p 2345:2345`) and connecting with `dlv connect :2345`. Build debug image with Delve included, or attach Delve to a running container. VS Code also supports Go debugging via Remote SSH extension.

### Q2: What is the difference between `--inspect` flag and attaching a debugger?
**A:** The `--inspect` flag (e.g., `--inspect=node`) makes container's internal debug protocol available to Chrome DevTools without code changes. Attaching a debugger (like Delve or VS Code) allows setting breakpoints and stepping through code. Choose based on your needs and workflow.

### Q3: How do you debug Python applications in containers?
**A:** Use Python's built-in debugger (pdb) by exposing debug port. Run `docker run -p 8000:8000 python -m pdb -m pdb.Pdb(addr=('0.0.0.0', 8000)) myapp.py` which starts a TCP debug server. Connect with `telnet localhost 8000` and use `(b)`, `(n)`, `(c)` commands to control execution.

### Q4: How do you debug Node.js applications remotely?
**A:** For simple cases, use Chrome DevTools with `--inspect` flag. For more advanced debugging, attach VS Code Remote Development extension or use `node --inspect`. Both allow setting breakpoints, stepping through code, and inspecting variables.

### Q5: What logging strategy should you use for debugging containerized applications?
**A:** Use structured logging (JSON format) with appropriate log levels (debug, info, warn, error). Send logs to external systems for aggregation. Use correlation IDs to trace requests across microservices. Don't log to stdout in production; use proper logging libraries.

### Q6: How do you troubleshoot performance issues in containers?
**A:** Monitor container resource usage with `docker stats`. Profile the application using language-specific tools (Go: pprof). Check for memory leaks and goroutine leaks. Use distributed tracing to identify bottlenecks across services.

### Q7: What are the security implications of exposing debug ports?
**A:** Debug ports should never be publicly accessible. Use SSH tunnels or VPN-restricted networks instead. Clean up debug containers when not in use. Set resource limits on debug containers to prevent them from consuming all resources. Audit debug access logs regularly.

### Q8: How do you debug in Kubernetes vs local Docker?
**A:** In Kubernetes, use `kubectl exec -it <pod> -- /bin/sh` to enter a container for ad-hoc debugging. For persistent debugging, use ephemeral debug containers that attach to the same pod. Use port-forwarding (`kubectl port-forward`) to access debug ports securely from your local machine.
