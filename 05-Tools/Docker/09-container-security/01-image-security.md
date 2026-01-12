---
---

## Summary
Container security ensures the integrity of the software supply chain. It involves scanning images for known vulnerabilities (CVEs), using minimal and trusted base images, running containers with least privilege, and signing images to verify their origin.

## Detailed Explanation

### Best Practices
1.  **Scanning**: Use tools like Trivy, Clair, or Docker Scan to check images against vulnerability databases.
2.  **Minimal Base Images**: Use `alpine` or `distroless`. Fewer packages = smaller attack surface.
3.  **Non-Root User**: By default, containers run as `root`. Create a user (e.g., `appuser`) and switch to it (`USER appuser`).
4.  **Content Trust (Signing)**: Use Docker Content Trust (Notary) to digitally sign images so nodes only run verified code.

### Distroless
Google's "Distroless" images contain *only* the application and its runtime dependencies. No shell (`sh`), no package manager (`apt`), no text editors. This makes debugging harder but exploitation much harder.

## Go-Specific Context/Examples

Go binaries are static. You don't need a full OS.

### Example: Multi-Stage Build (Secure)
```dockerfile
# Build Stage (Heavy)
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o myapp main.go

# Runtime Stage (Light & Secure)
FROM gcr.io/distroless/static-debian11
COPY --from=builder /app/myapp /
USER nonroot:nonroot
ENTRYPOINT ["/myapp"]
```

## Interview Questions

**Q: Why use `COPY` instead of `ADD` in Dockerfiles?**
**A:** `ADD` has extra features (unpacking tarballs, downloading URLs) which can be risky or unexpected. `COPY` just copies local files. Use `COPY` unless you explicitly need `ADD`'s features.

**Q: What is the risk of mounting `/var/run/docker.sock` inside a container?**
**A:** It gives the container full control over the Docker daemon on the host. A malicious container could use this to create new privileged containers, mount the host filesystem, and essentially gain root access to the host machine.

**Q: How does `distroless` improve security?**
**A:** If an attacker finds a Remote Code Execution (RCE) vulnerability in your Go app, they usually want to spawn a shell to explore/escalate (`/bin/sh`). In distroless, there is no shell. The attack is contained because there are no tools to use.
