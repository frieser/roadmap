---
---

## Summary
Docker Hub is the world's largest library and community for container images. It serves as the default registry where Docker looks for images if no other registry is specified (e.g., `docker pull nginx` implies `docker pull docker.io/library/nginx`).

## Detailed Explanation

### Repositories
*   **Public Repositories**: Images accessible by anyone. Open Source projects (official images) live here.
*   **Private Repositories**: Images accessible only to authorized users. Used by companies for proprietary code.

### Official Images
Images curated by Docker Inc. and upstream vendors (e.g., `golang`, `ubuntu`, `postgres`). They are scanned for vulnerabilities and follow best practices. They do not have a user prefix (e.g., just `python`, not `user/python`).

### Features
*   **Automated Builds**: Connects to GitHub/Bitbucket. Triggers an image build when code is pushed.
*   **Webhooks**: Triggers an external URL (e.g., a deployment pipeline) when a new image is pushed.

## Go-Specific Context/Examples

You can push your Go applications to Docker Hub easily.

### Example: Manual Push
```bash
# 1. Build
docker build -t myuser/my-go-app:v1 .

# 2. Login
docker login

# 3. Push
docker push myuser/my-go-app:v1
```

## Interview Questions

**Q: What is the `latest` tag?**
**A:** `latest` is just a default tag applied if no tag is specified. It is **not** dynamic or special. It points to whatever image was last pushed with the tag `latest`. Relying on it in production is dangerous because you don't know which version you are running. Always pin versions (e.g., `:v1.2.3` or sha256).

**Q: How do you authenticate with Docker Hub in CI/CD?**
**A:** Use an **Access Token** instead of your password. Tokens can have scoped permissions (Read/Write) and can be revoked easily. `docker login -u username -p $TOKEN`.

**Q: What is rate limiting on Docker Hub?**
**A:** Docker Hub limits anonymous pulls (e.g., 100 pulls per 6 hours) and authenticated free users (200 pulls). In a busy CI/CD environment or Kubernetes cluster, you can hit this limit quickly, causing `ErrImagePull`. The solution is to authenticate or use a pull-through cache/mirror.
