---
tags: ['docker', 'containers', 'cli', 'devops', 'tools', 'roadmap']
---

# Command Line Utilities in Docker

## Summary

Docker containers can package command-line utilities, providing consistent tool versions across environments without installing software on the host. This approach is useful for one-off commands, CI/CD pipelines, and accessing tools not available on the host OS. Containers can run utilities like `curl`, `jq`, `aws-cli`, `terraform`, and development tools in isolated environments.

## Detailed Explanation

### Running CLI Tools as Containers

```bash
# Pattern: docker run --rm <image> <command>

# Run a command and remove container when done
docker run --rm alpine cat /etc/os-release

# Interactive shell
docker run --rm -it ubuntu:22.04 bash

# Run with current directory mounted
docker run --rm -v $(pwd):/work -w /work alpine ls -la

# Option breakdown:
# --rm        Remove container after exit
# -it         Interactive with terminal
# -v          Mount volume
# -w          Set working directory inside container
```

### Common CLI Utilities

```bash
# CURL
docker run --rm curlimages/curl https://api.github.com

# JQ (JSON processor)
docker run --rm -i stedolan/jq '.' < data.json
echo '{"name":"test"}' | docker run --rm -i stedolan/jq '.name'

# AWS CLI
docker run --rm \
  -v ~/.aws:/root/.aws:ro \
  -v $(pwd):/work \
  -w /work \
  amazon/aws-cli s3 ls

# Azure CLI
docker run --rm -it \
  -v ${HOME}/.azure:/root/.azure \
  mcr.microsoft.com/azure-cli az login

# Google Cloud CLI
docker run --rm -it \
  -v ${HOME}/.config/gcloud:/root/.config/gcloud \
  gcr.io/google.com/cloudsdktool/cloud-sdk gcloud auth login

# Terraform
docker run --rm \
  -v $(pwd):/work \
  -w /work \
  hashicorp/terraform:latest init

docker run --rm \
  -v $(pwd):/work \
  -w /work \
  -e AWS_ACCESS_KEY_ID \
  -e AWS_SECRET_ACCESS_KEY \
  hashicorp/terraform:latest apply

# Ansible
docker run --rm \
  -v $(pwd):/work \
  -v ~/.ssh:/root/.ssh:ro \
  -w /work \
  willhallonline/ansible:latest ansible-playbook site.yml

# HTTPie (user-friendly curl alternative)
docker run --rm -it alpine/httpie https://api.github.com
```

### Development Tools

```bash
# Node.js / npm
docker run --rm \
  -v $(pwd):/app \
  -w /app \
  node:20 npm install

docker run --rm \
  -v $(pwd):/app \
  -w /app \
  node:20 npm test

# Python / pip
docker run --rm \
  -v $(pwd):/app \
  -w /app \
  python:3.11 pip install -r requirements.txt

docker run --rm \
  -v $(pwd):/app \
  -w /app \
  python:3.11 python script.py

# Go compiler
docker run --rm \
  -v $(pwd):/app \
  -w /app \
  golang:1.21 go build -o myapp .

# Rust compiler
docker run --rm \
  -v $(pwd):/app \
  -w /app \
  rust:latest cargo build --release

# Ruby / gem
docker run --rm \
  -v $(pwd):/app \
  -w /app \
  ruby:3.2 bundle install
```

### Creating Shell Aliases

```bash
# Add to ~/.bashrc or ~/.zshrc

# AWS CLI
alias aws='docker run --rm -it \
  -v ~/.aws:/root/.aws:ro \
  -v $(pwd):/work \
  -w /work \
  amazon/aws-cli'

# Terraform
alias terraform='docker run --rm -it \
  -v $(pwd):/work \
  -w /work \
  -e AWS_ACCESS_KEY_ID \
  -e AWS_SECRET_ACCESS_KEY \
  hashicorp/terraform:latest'

# jq
alias jq='docker run --rm -i stedolan/jq'

# Node.js
alias node='docker run --rm -it \
  -v $(pwd):/app \
  -w /app \
  node:20'

# Python
alias python='docker run --rm -it \
  -v $(pwd):/app \
  -w /app \
  python:3.11 python'

# Now use naturally:
# aws s3 ls
# terraform plan
# jq '.' file.json
# node script.js
```

### Shell Functions for Complex Tools

```bash
# For tools needing more configuration
# Add to ~/.bashrc or ~/.zshrc

# Terraform with state caching
tf() {
  docker run --rm -it \
    -v $(pwd):/work \
    -v terraform-plugins:/root/.terraform.d/plugins \
    -w /work \
    -e AWS_ACCESS_KEY_ID \
    -e AWS_SECRET_ACCESS_KEY \
    -e AWS_DEFAULT_REGION \
    hashicorp/terraform:1.5 "$@"
}

# kubectl with local config
kubectl() {
  docker run --rm -it \
    -v ~/.kube:/root/.kube:ro \
    -v $(pwd):/work \
    -w /work \
    bitnami/kubectl:latest "$@"
}

# Helm
helm() {
  docker run --rm -it \
    -v ~/.kube:/root/.kube:ro \
    -v ~/.helm:/root/.helm \
    -v $(pwd):/work \
    -w /work \
    alpine/helm:latest "$@"
}

# Docker-in-Docker for tools needing Docker
dind() {
  docker run --rm -it \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v $(pwd):/work \
    -w /work \
    "$@"
}
```

### CI/CD Pipeline Usage

```yaml
# GitLab CI example
image: docker:latest

stages:
  - lint
  - test
  - build
  - deploy

lint:
  image: hadolint/hadolint:latest
  script:
    - hadolint Dockerfile

test:
  image: python:3.11
  script:
    - pip install -r requirements.txt
    - pytest

build:
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t myapp:${CI_COMMIT_SHA} .
    - docker push myapp:${CI_COMMIT_SHA}

deploy:
  image: hashicorp/terraform:latest
  script:
    - terraform init
    - terraform apply -auto-approve
```

```yaml
# GitHub Actions example
name: CI
on: [push]

jobs:
  lint:
    runs-on: ubuntu-latest
    container: hadolint/hadolint:latest
    steps:
      - uses: actions/checkout@v4
      - run: hadolint Dockerfile

  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'myapp:latest'

  deploy:
    runs-on: ubuntu-latest
    container: 
      image: hashicorp/terraform:latest
    steps:
      - uses: actions/checkout@v4
      - run: terraform init && terraform apply -auto-approve
```

### Multi-Tool Containers

```dockerfile
# Dockerfile for custom CLI toolkit
FROM alpine:3.18

# Install common tools
RUN apk add --no-cache \
    curl \
    jq \
    yq \
    git \
    openssh-client \
    bash \
    vim \
    less \
    bind-tools \
    netcat-openbsd \
    httpie

# Install kubectl
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
    chmod +x kubectl && mv kubectl /usr/local/bin/

# Install helm
RUN curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Install aws-cli
RUN apk add --no-cache python3 py3-pip && \
    pip3 install awscli --break-system-packages

WORKDIR /work
CMD ["/bin/bash"]
```

```bash
# Build and use
docker build -t my-toolkit .
docker run --rm -it \
  -v $(pwd):/work \
  -v ~/.aws:/root/.aws:ro \
  -v ~/.kube:/root/.kube:ro \
  my-toolkit

# Now have all tools available in one container
```

### Security Considerations

```bash
# Mount credentials read-only
docker run --rm \
  -v ~/.aws:/root/.aws:ro \
  amazon/aws-cli s3 ls

# Use environment variables instead of files
docker run --rm \
  -e AWS_ACCESS_KEY_ID="$AWS_ACCESS_KEY_ID" \
  -e AWS_SECRET_ACCESS_KEY="$AWS_SECRET_ACCESS_KEY" \
  amazon/aws-cli s3 ls

# Avoid running as root when possible
docker run --rm \
  --user $(id -u):$(id -g) \
  -v $(pwd):/work \
  -w /work \
  node:20 npm install

# Don't mount Docker socket unless necessary
# Mounting /var/run/docker.sock gives full Docker access!
```

## Interview Questions

### Q1: Why would you run CLI tools in Docker containers?
**A:** For consistent tool versions across environments, avoiding installation on host systems, running tools not available for your OS, and ensuring CI/CD pipelines use the same tool versions as development.

### Q2: What does the `--rm` flag do and why use it for CLI containers?
**A:** `--rm` automatically removes the container when it exits. CLI containers are typically one-off tasks, so removing them prevents accumulation of stopped containers and saves disk space.

### Q3: How do you give a containerized CLI tool access to local files?
**A:** Mount the current directory as a volume: `-v $(pwd):/work -w /work`. This makes local files accessible inside the container and changes persist on the host.

### Q4: How do you pass credentials to CLI tools in containers?
**A:** Either mount credentials read-only (`-v ~/.aws:/root/.aws:ro`) or use environment variables (`-e AWS_ACCESS_KEY_ID`). Environment variables are often more secure and flexible for CI/CD.

### Q5: Why might you create shell aliases for Dockerized tools?
**A:** Aliases make containerized tools feel native. Instead of typing the full `docker run ...` command, you can just type `terraform plan`. This improves developer experience while maintaining container benefits.

### Q6: What is the purpose of `-v terraform-plugins:/root/.terraform.d/plugins`?
**A:** This creates a named volume to cache downloaded Terraform plugins across container runs. Without this, plugins would re-download on every run, wasting time and bandwidth.

### Q7: Why use containers for CI/CD pipeline steps?
**A:** Containers ensure reproducible builds with exact tool versions, provide isolation between jobs, make pipelines portable across CI systems, and allow easy tool updates by changing image tags.

### Q8: What are the security implications of mounting `/var/run/docker.sock`?
**A:** Mounting the Docker socket gives the container full control over Docker on the host - it can start/stop any container, access any volume, and potentially escape to the host. Only do this when absolutely necessary and never with untrusted images.
