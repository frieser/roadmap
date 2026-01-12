---
---

# Bitbucket

Bitbucket (by Atlassian) is heavily favored in enterprise environments using Jira and Confluence. Its strength lies in deep **integration** with project management tools rather than standalone features.

## Key Features

### 1. Bitbucket Pipelines
CI/CD integrated into the UI.
- **bitbucket-pipelines.yml**: Configuration file.
- **Pipes**: Reusable, Docker-based tasks (similar to GitHub Actions).
  - Example: `atlassian/aws-s3-deploy`, `sonarsource/sonarcloud-scan`.
- **Steps**: Run in isolated Docker containers.

### 2. Jira Integration ("Smart Commits")
Control Jira issues from Git commit messages.
- Syntax: `KEY-123 #command`
- Examples:
  - `PROJ-101 #comment Fixed NPE in login handler`
  - `PROJ-101 #in-progress`
  - `PROJ-101 #done`

### 3. Code Insights
Brings quality reports into the Pull Request view.
- **Annotations**: Inline flags on specific code lines (e.g., "SQL Injection detected here").
- **Quality Gates**: Block merges if coverage drops or critical bugs are found.

## Backend Usage

### Pipeline with Pipes
Example deployment using pipes to abstract complexity.

```yaml
pipelines:
  default:
    - step:
        name: Build and Test
        image: golang:1.24
        script:
          - go test ./...
  branches:
    master:
      - step:
          name: Deploy to AWS
          script:
            - pipe: atlassian/aws-serverless-deploy:1.0.0
              variables:
                AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY
                AWS_SECRET_ACCESS_KEY: $AWS_SECRET_KEY
                STACK_NAME: 'my-backend-stack'
```

## Interview Questions
1. **What are Bitbucket Pipes?**
   - Pre-built Docker containers that encapsulate complex CI/CD logic (deployments, notifications) to keep YAML clean.
2. **How do Smart Commits help?**
   - They synchronize code and tracking, eliminating the need to manually update Jira tickets after pushing code.
3. **Report vs Annotation?**
   - *Report*: High-level status (Pass/Fail) of a tool run.
   - *Annotation*: Specific finding attached to a line of code in the diff.
