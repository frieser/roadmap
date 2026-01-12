---
tags: ['tools', 'roadmap']
---

## Summary
**Run Tasks** in Terraform Cloud and Enterprise allow you to integrate third-party services directly into the Terraform run lifecycle. They act as external "hooks" that can perform security scans, compliance checks, or cost analysis (e.g., using Snyk, Infracost, or Bridgecrew) at specific points in the run, either providing advice or blocking the run if mandatory requirements are not met.

## Detailed Explanation

### 1. Integration Points
Run tasks can be attached to different stages of a run:
- **Pre-plan**: Executes before the plan begins. Useful for checking prerequisites or system availability.
- **Post-plan**: The most common point. Executes after the plan is generated. The task receives the plan in JSON format to analyze what infrastructure is being added, changed, or destroyed.
- **Pre-apply**: Executes after a plan is approved but before changes are made.

### 2. Enforcement Levels
- **Advisory**: The run continues even if the task fails. A warning is shown in the UI.
- **Mandatory**: The run is blocked and cannot proceed (e.g., cannot Apply) if the task returns a failure.

### 3. Partner Ecosystem
HashiCorp partners provide pre-built Run Tasks for various domains:
- **Security**: Snyk, Bridgecrew, Prisma Cloud (scans for vulnerabilities).
- **Cost**: Infracost (checks if the plan exceeds a budget).
- **Compliance**: ServiceNow (validates changes against a change management ticket).

### 4. Custom Run Tasks
You can build your own Run Task by creating a web service that accepts a JSON payload from TFC, processes it, and returns a status (passed/failed) via a callback URL.

### HCL Configuration Example

**Registering and Attaching a Run Task:**
```hcl
# 1. Define the Run Task at the Organization level
resource "tfe_organization_run_task" "snyk_scan" {
  organization = "my-org"
  name         = "Snyk-Security-Check"
  url          = "https://api.snyk.io/v1/terraform-cloud/webhook/..."
  hmac_key     = var.snyk_hmac # Secret for payload verification
  enabled      = true
}

# 2. Attach the task to a specific Workspace
resource "tfe_workspace_run_task" "api_enforcement" {
  workspace_id      = tfe_workspace.api_prod.id
  task_id           = tfe_organization_run_task.snyk_scan.id
  enforcement_level = "mandatory" # Blocks apply on failure
  stage             = "post_plan"
}
```

### Go Application
Custom Run Task services are often written in Go due to its excellent JSON handling and concurrency model for processing multiple webhooks.

```go
package main

import (
	"encoding/json"
	"net/http"
)

type RunTaskPayload struct {
	PlanJSONURL string `json:"plan_json_url"`
	CallbackURL  string `json:"task_result_callback_url"`
}

func handler(w http.ResponseWriter, r *http.Request) {
	var payload RunTaskPayload
	json.NewDecoder(r.Body).Decode(&payload)

	// 1. Fetch Plan JSON from payload.PlanJSONURL
	// 2. Perform custom validation logic
	// 3. POST result back to payload.CallbackURL
	
	w.WriteHeader(http.StatusAccepted)
}

func main() {
	http.HandleFunc("/run-task", handler)
	http.ListenAndServe(":8080", nil)
}
```

## Interview Questions

**Q: What is a Run Task in Terraform Cloud?**
**A:** A Run Task is an integration that allows TFC to communicate with external 3rd-party tools (like Snyk or Infracost) during the run lifecycle. It sends plan data to the tool and waits for a "pass" or "fail" response before proceeding.

**Q: How does a Run Task differ from Sentinel or OPA?**
**A:** Sentinel and OPA are internal policy engines that execute *within* the TFC environment using specific policy languages. Run Tasks are *external* calls to other services, allowing you to use tools that aren't natively part of the Terraform ecosystem.

**Q: What happens if a "Mandatory" Run Task fails?**
**A:** The Terraform run is halted. If it fails at the `post_plan` stage, the user will be unable to "Apply" the changes until the issues are fixed or the enforcement level is changed.

**Q: At what stage would you typically attach a cost estimation tool?**
**A:** Usually at the `post_plan` stage, because the tool needs to see the final list of resources being created or modified (the plan output) to accurately calculate the cost impact.
