# DevOps Summary — 19 Sub-Topics

## OS Foundations
**Windows**: PowerShell (object pipeline), WSL2, WinRM/SSH, DSC, NTFS ACLs, Registry. `.NET` eco.
**Unix/Linux**: Debian/Ubuntu (APT), RHEL/Rocky/Alma (DNF, SELinux, Kickstart), SLES (Zypper, Btrfs/Snapper, YaST). BSD: FreeBSD (ZFS, Jails, Ports), OpenBSD (pf, pledge, unveil), NetBSD (portability, pkgsrc, Rump Kernels).

| Distro | Pkg Mgr | FS | Key Feature |
|--------|---------|-----|-------------|
| Debian | APT/dpkg | ext4 | Stability, cloud standard |
| RHEL | DNF | xfs | SELinux, enterprise compliance |
| SLES | Zypper | Btrfs | Snapper rollbacks, YaST |

**Terminal**: Process (ps/top/htop/pgrep), Performance (vmstat/iostat/sar/free/lsof), Networking (ping/ss/dig/tcpdump/curl), Text (grep/sed/awk/cut/jq).
**Scripting**: Bash (set -euo pipefail), PowerShell (cmdlets, objects).
**Editors**: Vim (modal, ubiquitous), Nano (simple), Emacs (extensible, Lisp). Common: `gopls` LSP.

## VCS & Hosting
**Git**: DAG of Blobs/Trees/Commits, `go-git` for pure-Go manipulation, bisect, worktrees, hooks. GitOps: Git as single source of truth.
**Hosting**: GitHub (Actions, Dependabot, CodeQL, Checks API), GitLab (CI/CD, Auto DevOps, DAG pipelines), Bitbucket (Pipelines, Jira integration, Pipes).

## Containers
**Docker**: Namespaces (PID/NET/MNT), Cgroups, UnionFS (OverlayFS). Multi-stage builds, rootless mode.
**LXC/LXD**: System containers (full OS), longer-lived, pet vs cattle.
**containerd**: CRI-compliant runtime, runc (OCI), CNI networking.

| Feature | Docker | LXC | containerd |
|---------|--------|-----|------------|
| Focus | App containers | System containers | Runtime lifecycle |
| Persistence | Ephemeral | Persistent | Configurable |
| K8s compat | Via dockershim (legacy) | Via CRI | Native CRI |

## Networking
**OSI**: L3 (IP/routing), L4 (TCP/UDP ports, NLBs), L7 (HTTP/DNS, ALBs/Ingress).
**Protocols**: DNS (A/AAAA/CNAME/MX/TXT, TTL, recursive vs iterative), HTTP (methods, status codes, headers), HTTPS (TLS handshake, certificates, HSTS), SSL/TLS (mTLS, PFS, X.509), SSH (pubkey auth, tunneling), FTP/SFTP (vs SFTP = SSH-based).
**Email**: SMTP (push, port 25/587), IMAP (sync, port 143/993), POP3 (download, port 110/995). SPF (IP allowlist), DKIM (signature), DMARC (policy + reporting).

## Web Servers
| Server | Lang | Arch | Strength |
|--------|------|------|----------|
| Nginx | C | Event-driven (epoll) | Reverse proxy, ingress, high-concurrency |
| Caddy | Go | Event-driven (goroutines) | Auto HTTPS, memory-safe, simple |
| Apache | C | MPM (Prefork/Worker/Event) | .htaccess, mod_php, legacy |
| Tomcat | Java | Servlet container | Java/WAR apps |
| IIS | C++ | Windows-native | .NET, App Pools, AD integration |

## Cloud Providers
**AWS**: EC2, S3, RDS, VPC, IAM (roles vs users), Security Groups (stateful) vs NACLs (stateless).
**Azure**: Resource Groups, AKS, Azure AD/Entra ID, ARM/Bicep, Blob Storage.
**GCP**: GKE (gold standard), Global VPC, BigQuery, Preemptible VMs.
**DigitalOcean**: Droplets, Spaces (S3-compat), App Platform. Simple pricing.
**Heroku**: PaaS, Dynos, Buildpacks, Procfile, 12-Factor App, ephemeral FS.

## Serverless
| Platform | Runtime | Trigger Model | Cold Start Mitigation |
|----------|---------|---------------|-----------------------|
| AWS Lambda | Go/Node/Python | Event-driven (API GW, S3, SQS) | Provisioned Concurrency |
| GCF | Go via Functions Framework | Eventarc, Gen2 on Cloud Run | Concurrency per instance |
| Azure Functions | Go via Custom Handler | Triggers + Bindings | Premium Plan (pre-warmed) |
| Vercel/Netlify | Go/Node via API dir | Git push, Edge vs Serverless | Edge functions (near-zero) |

## Config Management & Provisioning
| Tool | Model | Lang | Architecture |
|------|-------|------|-------------|
| Ansible | Push | YAML | Agentless (SSH/WinRM) |
| Chef | Pull | Ruby (DSL) | Agent + Server (Cookbooks) |
| Puppet | Pull | Declarative DSL | Agent + Master (Catalog/DAG) |
| Terraform | IaC | HCL | Plan/Apply, State file, Providers |
| Pulumi | IaC | Go/TS/Python | Real languages, Stacks, Automation API |
| CloudFormation | IaC | JSON/YAML | AWS-native, Change Sets, Drift Detection |
| AWS CDK | IaC | Go/TS/Python | Constructs, Synth → CloudFormation |

## CI/CD Tools
**Jenkins**: Controller-Agent, Jenkinsfile (Declarative/Scripted Groovy), 1800+ plugins.
**GitHub Actions**: YAML workflows, matrix builds, OIDC auth, reusable Actions.
**GitLab CI**: .gitlab-ci.yml, DAG (needs), Auto DevOps, Review Apps.
**CircleCI**: config.yml, Orbs, parallelism, test splitting.

**Rollback Strategies**: Rolling (slow, no extra infra), Blue/Green (instant, double infra), Canary (gradual, complex).

## Observability

| Category | Tools |
|----------|-------|
| Logs | Splunk (SPL, HEC, Indexer clustering), ELK (Elasticsearch, Logstash, Kibana, Beats), Graylog (GELF, Streams, Pipelines) |
| Infra Metrics | Prometheus (Pull, PromQL, TSDB, Alertmanager), Nagios (Check-based, NRPE), Zabbix (Agent/Proxy, SNMP, LLD), Monit (Local watchdog), Datadog (Push, DogStatsD, Tags) |
| App Monitoring | Datadog APM (Auto-instrumentation, Traces), New Relic (Apdex, Transaction tracing), AppDynamics (Business Transactions, Flow Maps), Jaeger (Distributed tracing, DAG), OpenTelemetry (Vendor-neutral, 3 pillars, OTel Collector) |
| Visualization | Grafana (multi-source, dashboards, Loki for logs, Tempo for traces) |

## Secret Management
| Tool | Storage | Key Mgmt | Best For |
|------|---------|----------|----------|
| HashiCorp Vault | Centralized server | Identity-based, dynamic secrets | Multi-cloud, on-prem, enterprise |
| AWS Secrets Manager | AWS-hosted | IAM, auto-rotation | AWS-native workloads |
| SOPS | Encrypted in Git | KMS, PGP, age | GitOps, IaC files |
| Sealed Secrets | Encrypted in Git (K8s) | In-cluster RSA | Pure K8s GitOps |

## Artifact Management
**JFrog Artifactory / Sonatype Nexus**: Local (internal builds), Remote (proxied cache), Virtual (aggregated URL). GOPROXY integration, Docker registry, format-specific metadata.

## GitOps

| Tool | Arch | State | Secret Integration | UI |
|------|------|-------|-------------------|-----|
| ArgoCD | Centralized controller | Own DB (Redis) | Vault Plugin, Sealed Secrets | Rich Web UI + CLI |
| FluxCD | Modular Toolkit (multi-controller) | K8s etcd | Native SOPS | CLI-centric |

## Service Mesh

| Mesh | Proxy | Complexity | Resource Usage | Key Feature |
|------|-------|------------|----------------|-------------|
| Istio | Envoy (C++) | High | High | Ambient Mesh, fine-grained traffic mgmt |
| Linkerd | linkerd2-proxy (Rust) | Low | Very low | Auto mTLS, simplicity |
| Consul | Envoy | Medium | Medium | Multi-platform (K8s+VMs), Intentions |

## Cloud Design Patterns
**Availability**: Circuit Breaker (Closed→Open→Half-Open), Health Endpoint Monitoring, Throttling.
**Data**: CQRS (separate read/write models), Event Sourcing, Sharding.
**Structural**: Sidecar (logging/monitoring proxy), Ambassador (outbound proxy), Adapter (interface standardizer).
**Reliability**: Retry + Exponential Backoff + Jitter, Bulkhead (resource pools/goroutine semaphores).
**Migration**: Strangler Fig (incremental replacement via routing facade).
