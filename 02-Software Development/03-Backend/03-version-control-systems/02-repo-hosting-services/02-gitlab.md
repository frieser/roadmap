---
---

# GitLab

GitLab is a complete **DevSecOps platform** delivered as a single application. It is widely used in enterprise environments, often self-hosted, offering deep control over the CI/CD pipeline and infrastructure.

## Key Features

### 1. GitLab CI/CD
Defined in `.gitlab-ci.yml`, known for its flexibility.
- **Stages**: Sequential phases (Build -> Test -> Deploy).
- **Artifacts**: Files passed between stages (binaries, reports).
- **Cache**: Dependencies stored to speed up future runs (node_modules, GOCACHE).
- **Runners**:
    - **Shared**: Available to all projects.
    - **Specific**: Assigned to specific projects/groups (critical for backend access to private DBs).

### 2. Kubernetes Agent
Secure, two-way integration with K8s clusters.
- **Push-based**: Pipeline runs `kubectl` commands via the Agent.
- **Pull-based (GitOps)**: Agent watches the repo and applies manifests to the cluster.
- Replaces the deprecated certificate-based integration.

### 3. GitLab Flow
A simplified branching strategy suited for continuous deployment.
- **Feature Branches**: Merge to `main`.
- **Environment Branches**: Code merges to `production` or `pre-prod` branches to trigger deployments.

## Backend Configuration

### Go Pipeline Example
Optimized for Go with caching and docker builds.

```yaml
image: golang:1.24

variables:
  GOPATH: $CI_PROJECT_DIR/go
  REPO_NAME: gitlab.com/org/project

# Cache Go modules between builds
cache:
  paths:
    - go/pkg/mod/

stages:
  - test
  - build

test:
  stage: test
  script:
    - mkdir -p $GOPATH/src/$REPO_NAME
    - ln -s $CI_PROJECT_DIR $GOPATH/src/$REPO_NAME
    - cd $GOPATH/src/$REPO_NAME
    - go test -race ./...

build_image:
  stage: build
  image: docker:24
  services:
    - docker:dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

## Interview Questions
1. **Artifacts vs Cache?**
   - *Cache*: For dependencies (download once, reuse). Temporary.
   - *Artifacts*: For build outputs (binaries, jars). persistent, passed to next stages.
2. **GitLab Agent vs Cert-based K8s?**
   - Agent is secure (tunnel), doesn't expose K8s API to internet, and supports GitOps natively.
3. **What is Auto DevOps?**
   - Pre-built pipelines that automatically detect language, build, test, and deploy using Herokuish buildpacks and Helm.
