---
tags: ['tools', 'roadmap']
---

## Summary
Terraform Enterprise and HCP Terraform (formerly Terraform Cloud) offer advanced governance and compliance features designed for large-scale infrastructure management. Key features include **Sentinel** and **OPA** for Policy-as-Code, **Cost Estimation** for budget visibility, **Drift Detection** for maintaining infrastructure integrity, and the **Private Module Registry** for standardizing reusable infrastructure components across the organization.

## Detailed Explanation

### 1. Policy as Code: Sentinel and OPA
Governance in Terraform is achieved through policies that are evaluated during the run lifecycle.
- **Sentinel**: HashiCorp's functional policy language. It allows fine-grained control, such as restricting instance types, mandating tags, or limiting deployment windows.
- **Open Policy Agent (OPA)**: A CNCF graduated project using the **Rego** language. HCP Terraform supports OPA, allowing teams to use a industry-standard policy engine for infrastructure governance.

**Enforcement Levels:**
- **Advisory**: Warning only; does not block runs.
- **Soft Mandatory**: Blocks runs but allows authorized overrides.
- **Hard Mandatory**: Blocks runs and requires a policy change or exception to proceed.

### 2. Cost Estimation
Before infrastructure is applied, Terraform calculates the estimated monthly cost based on the plan. 
- **Coverage**: Supports AWS, Azure, and Google Cloud.
- **Integration**: Works with Sentinel/OPA to block deployments that exceed a specific budget delta.

### 3. Drift Detection and Continuous Validation
These features move Terraform from a "deployment tool" to a "post-deployment monitoring tool."
- **Drift Detection**: Periodically scans the state to find differences between the real-world infrastructure and the Terraform configuration (e.g., manual "ClickOps" changes).
- **Continuous Validation**: Uses `check` blocks and `postcondition` assertions to verify that infrastructure remains healthy over time (e.g., checking SSL certificate expiration or API reachability).

### 4. Private Module Registry (PMR)
A centralized repository for internal teams to share and version approved infrastructure modules.
- **No-Code Provisioning**: Allows platform teams to publish modules that non-technical users can deploy via a GUI form without writing HCL.
- **Module Testing**: Automatically runs `terraform test` on new module versions to ensure reliability.

### HCL and Policy Examples

**Sentinel Policy (Restricting AWS Instance Types):**
```sentinel
import "tfplan/v2" as tfplan

allowed_types = ["t2.micro", "t3.micro"]

main = rule {
    all tfplan.resource_changes as _, rc {
        rc.type is "aws_instance" and rc.mode is "managed" implies
        rc.change.after.instance_type in allowed_types
    }
}
```

**Continuous Validation (HCL Check Block):**
```hcl
check "health_check" {
  data "http" "service" {
    url = "https://api.myapp.com/health"
  }

  assert {
    condition     = data.http.service.status_code == 200
    error_message = "App health check failed with status ${data.http.service.status_code}"
  }
}
```

### Go Application
Developers often use the `go-tfe` SDK to programmatically manage these enterprise features, such as listing policy sets or managing modules in the PMR.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/hashicorp/go-tfe"
)

func main() {
	config := &tfe.Config{
		Token: "YOUR_TFE_TOKEN",
	}
	client, err := tfe.NewClient(config)
	if err != nil {
		log.Fatal(err)
	}

	// Example: List all policy sets in an organization
	policySets, err := client.PolicySets.List(context.Background(), "my-org", &tfe.PolicySetListOptions{})
	if err != nil {
		log.Fatal(err)
	}

	for _, ps := range policySets.Items {
		fmt.Printf("Policy Set: %s, Enforcement Level: %s\n", ps.Name, ps.Global)
	}
}
```

## Interview Questions

**Q: What is the difference between Sentinel and OPA in HCP Terraform?**
**A:** Sentinel is HashiCorp's proprietary functional language designed specifically for the HashiCorp stack with deep integration. OPA is an open-source, vendor-neutral policy engine using the Rego language, allowing organizations to standardize policies across multiple tools beyond just Terraform.

**Q: How does Drift Detection help in a production environment?**
**A:** It identifies "ClickOps" changes—manual modifications made directly in the cloud console—that bypass the version-controlled Terraform code. This ensures the code remains the single source of truth and prevents configuration drift that could lead to deployment failures.

**Q: What are the three enforcement levels for policies?**
**A:** Advisory (log only), Soft Mandatory (blocks but allows override), and Hard Mandatory (blocks and requires fix).

**Q: How does "No-Code Provisioning" work?**
**A:** Platform engineers publish a module to the Private Module Registry and mark it as "no-code ready." Developers can then select the module in the UI, fill out a variable form, and TFC handles the workspace creation and deployment without the developer writing HCL.
