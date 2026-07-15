### Containers
| Concept | Detail |
|----------|--------|
| Definition | Lightweight user-space isolation on shared kernel |
| vs VM | No hypervisor, near-native perf (1-5% overhead) |
| Isolation | Linux namespaces: PID, NET, MNT, UTS, IPC, USER, CGROUP, TIME |
| Resource | cgroups v2: cpu.max, memory.max, memory.high, pids.max |
| Filesystem | OverlayFS: stacked R/O layers + writable top (CoW) |

### OCI & Runtimes | OCI: runtime (`runc`), image (manifest+config+layers), distribution specs | Docker: CLI→dockerd→containerd→runc→kernel | Alts: Podman (daemonless/rootless), CRI-O, Buildah, Skopeo; K8s killed dockershim v1.24
### Registries | Hub: rate-limited | GHCR: PAT, GH Actions | ECR: IAM | GAR: vuln-scan | ACR: Azure AD | Harbor: self-host + Trivy
### Tagging | Never `:latest`; pin digest `@sha256:...` or semver `:v1.2.3`; promote: SHA→dev→staging→prod
### Dockerfile | `FROM` multi-stage; `COPY` > `ADD`; exec form; `ENTRYPOINT`+`CMD`; clean same `RUN`; `.dockerignore`
### Caching | Order least→most volatile; `COPY pkg.json` before `COPY .`; BuildKit `--mount=type=cache`; CI `--cache-from/to`

### Persistent Data | Writable layer: lost on `rm` | Named vol: persists in `/var/lib/docker/volumes/` | Bind `-v $(pwd):/app`: host-dependent | tmpfs: lost on stop
### CLI | `image ls/prune/rmi`, `ps -a/stop/rm -f/logs -f`, `volume create/ls/prune/rm`, `network create/ls/connect/disconnect`, `exec -it <c> sh`
### Runtime | `-p H:P`, `-e`, `--env-file`, `--memory --cpus`, `--restart unless-stopped`, `--read-only --tmpfs /tmp`, `--user $(id -u)`, `--health-cmd/interval`

### Security Layers
| Layer | Mechanism |
|-------|-----------|
| Base | Alpine > slim > distroless > scratch |
| User | `USER nonroot`, `--user $(id -u):$(id -g)` |
| Caps | `--cap-drop=ALL --cap-add=NET_BIND_SERVICE` |
| Syscall | seccomp `docker-default` |
| MAC | AppArmor/Ubuntu, SELinux/RHEL |
| UID | Rootless mode, user namespaces |
| Scan | Trivy/Grype/Scout; gate CI on CRITICAL |
| Secrets | BuildKit `--secret`; env at runtime; never in image |

### DevEx | Hot reload: bind mount + nodemon/air/watchdog; debug: Delve `:2345`, pdb TCP, Node `--inspect`; test: `--rm <img> <cmd>`; CI: GH `build-push-action`, GitLab `docker:dind`

### Orchestration
| Tool | When |
|------|------|
| Compose | Single-host dev/staging |
| Swarm | Native cluster; `stack deploy`, routing mesh, Raft quorum |
| Nomad | HCL jobs; simpler than K8s, HashiCorp |
| Kubernetes | Pods/Deployments/Services/Ingress; Go `client-go` |
| PaaS (Cloud Run, Fargate) | Serverless; scale-to-zero; cold starts |
