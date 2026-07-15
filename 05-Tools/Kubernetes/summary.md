### 01-intro | 03-setup | 04-running-apps
- `01-overview`, `02-why-use`, `03-key-concepts` — CP: API, etcd, Scheduler, CM. Workers: Kubelet, Kube-proxy, Runtime. Pod = smallest unit. Declarative model, self-healing, Go synergy. etcd = Raft-backed store. `04-alternatives` — Swarm, Nomad, ECS, Fargate, Cloud Run, OpenShift.
- `01-deploying-first-app` — Containerize→Deployment YAML→`kubectl apply`→Service. Multi-stage Docker for Go. Readiness/Liveness probes.
- `02-managed-providers` — GKE Autopilot, EKS Fargate, AKS Automatic. CP ~$0.10/hr. Shared responsibility. `03-local-cluster` — Kind (Go, CI), Minikube, k3d.
- `01-pods`, `02-replicasets`, `03-deployments` — Shared net/storage, localhost. RS: self-healing, set-based selectors. Deployments: RollingUpdate (default), `maxSurge`/`maxUnavailable`, `rollout undo`, 10 revisions.
- `04-statefulsets`, `05-jobs` — Stable identity (`name-0`), PVC per pod, Headless Service. Jobs: `parallelism`/`completions`/`backoffLimit`/`ttlSecondsAfterFinished`.

### 05-config | 06-networking
- `01-configmaps` — Env (static), volumes (live), CLI args. 1MB, `subPath` blocks auto-update. `02-secrets` — Opaque/TLS/dockerconfigjson. Base64 NOT encrypted. CSI driver for Vault/AWS.
- `01-external-access` — ClusterIP→NodePort→LoadBalancer→Ingress. SSL at Ingress. `02-load-balancing` — L4: iptables/IPVS. L7: Gateway API. gRPC: headless+client LB.
- `03-pod-to-pod` — Flat network (every Pod gets IP). CNI: overlay (Flannel) vs direct (Calico/Cilium). CoreDNS.

### 07-security | 08-resources
- `01-rbac` — Role/ClusterRole+Binding. Additive only. `02-network-policies` — podSelector, Ingress/Egress. Requires Calico/Cilium.
- `03-container-security` — SecurityContext: runAsNonRoot, readOnlyRootFS, drop ALL. PSS: Privileged/Baseline/Restricted. PSA via labels.
- `01-requests-limits` — CPU: throttled. Mem: OOMKilled. QoS: Guaranteed>Burstable>BestEffort. Go: `GOMEMLIMIT`, `automaxprocs`.
- `02-quotas` — ResourceQuota (namespace agg) + LimitRange (per-container). Admission check only. `03-optimization` — Metrics Server→VPA, pprof, PGO.

### 09-observability | 10-storage | 11-scheduling
- `01-three-pillars` — Logs→Fluentd/Loki, Metrics→Prometheus, Traces→OTel→Jaeger. `02-probes` — Liveness (restart), Readiness (endpoints), Startup (gate).
- `03-engines` — Prometheus pull/PromQL, Grafana, ELK, Jaeger. OTel Collector. Cardinality danger.
- `01-csi-drivers` — gRPC Identity/Controller/Node. Sidecars. Dynamic provisioning. `02-stateful-apps` — StatefulSet+PVC+Headless Service. Graceful shutdown.
- `01-basics` — Filter→Score→Bind. `02-taints` — NoSchedule/NoExecute. `03-topology-spread` — `maxSkew`/`topologyKey`. `04-priorities`/`05-evictions` — PriorityClass→preempt. QoS eviction order.

### 12-autoscaling | 13-deployment-patterns
- `01-hpa` — `ceil[current*(current/desired)]`. Custom metrics. `02-vpa` — Recommender/Updater/Admission. Off/Initial/Auto. `03-cluster-autoscaler` — CA (node groups) vs Karpenter (direct API, consolidation).
- `01-ci-cd` — Build→test→push. ArgoCD pull model. `02-gitops` — Declarative, versioned, pulled, reconciled. ArgoCD vs Flux. Drift detection.
- `03-helm` — Chart+templates+values. Go `text/template`. No Tiller (v3). `04-canary` — Flagger+Istio, weighted shift, auto-analysis.
- `05-blue-green` — Dual Deployments, Service switch. `06-rolling-updates` — `maxSurge`/`maxUnavailable`. Graceful shutdown critical.
- `07-kustomize` — Template-free. Base+Overlays. `kubectl -k`. ConfigMap hash trigger. `08-sidecar` — `initContainers`+`restartPolicy:Always` (v1.33 stable).
- `09-operators` — CRD+Controller. Reconciler: observe→analyze→act. Kubebuilder, controller-runtime. Day-2 automation.

### 14-advanced | 15-cluster-ops
- `01-custom-controllers` — Informers, Listers, Workqueues. Level-triggered. `02-custom-schedulers` — Framework (Go plugins) vs Extenders (HTTP).
- `03-crds` — OpenAPI v3 validation, `/status` subresource, conversion webhooks, finalizers. `04-extensions-apis` — API Aggregation, Mutating→Validating webhooks.
- `01-own-cluster` — Managed vs Self. etcd = biggest risk. `02-control-plane` — kubeadm: preflight→certs→static Pods. Stacked/External etcd. HA: 3+ CP+LB.
- `03-worker-nodes` — kubeadm join, cordon vs drain, CNI required. `04-multi-cluster` — Hub-and-spoke (GitOps), CAPI, Istio mesh.

### Go cross-cutting
- `client-go`: `InClusterConfig()` / `BuildConfigFromFlags()`. Probes: `/healthz`, `/readyz` → `srv.Shutdown(ctx)` on SIGTERM.
- `GOMEMLIMIT` (Go 1.19+) + `automaxprocs` for container-aware runtime. Multi-stage Docker → distroless/static:nonroot.
- `kubebuilder` markers: `+rbac`, `+subresource:status`, `+validation`. Reconciler pattern: Get CR → diff spec/status → act.
