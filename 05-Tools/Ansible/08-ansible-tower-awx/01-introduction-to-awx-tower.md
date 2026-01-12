---
---

## Summary
Ansible Tower (Red Hat product) and AWX (Open Source upstream) provide a web-based user interface, REST API, and task engine for Ansible. They solve the problem of "Scaling Ansible" by centralizing execution, managing credentials securely, adding Role-Based Access Control (RBAC), and keeping audit logs.

## Detailed Explanation

### Key Features
1.  **Centralized Execution**: No more "It worked on my laptop." Playbooks run in a consistent environment inside the Tower/AWX cluster.
2.  **RBAC**: Control who can run what. Give "Junior Devs" permission to restart webservers (Run Job), but not to read the SSH keys (Edit Credential).
3.  **Credentials**: Store SSH keys, AWS secrets, and Vault passwords securely. They are injected into playbooks at runtime but never exposed to the UI user.
4.  **API**: Everything in the UI is available via REST API, allowing other tools (ServiceNow, Jenkins, Slack) to trigger automation.

### Architecture
*   **Web**: Django-based UI/API.
*   **Task**: Workers that actually run `ansible-playbook`.
*   **RabbitMQ/Redis**: Message queue between web and workers.
*   **PostgreSQL**: Database for storing job history and inventory.

## Go-Specific Context/Examples

Since AWX exposes a full REST API, you can write Go tools to trigger jobs.

### Example: Triggering a Job via API in Go

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"net/http"
)

func main() {
	url := "https://awx.example.com/api/v2/job_templates/15/launch/"
	token := "Bearer YOUR_OAUTH_TOKEN"

	req, _ := http.NewRequest("POST", url, bytes.NewBuffer([]byte("{}")))
	req.Header.Set("Authorization", token)
	req.Header.Set("Content-Type", "application/json")

	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		fmt.Println("Error launching job:", err)
		return
	}
	defer resp.Body.Close()

	fmt.Println("Job Launched, Status:", resp.Status)
}
```

## Interview Questions

**Q: What is the relationship between AWX and Ansible Tower?**
**A:** AWX is the upstream open-source project (like Fedora). It gets new features fast but is less stable. Ansible Tower (now Automation Controller) is the downstream enterprise product supported by Red Hat (like RHEL), based on stable AWX releases.

**Q: Why use AWX instead of just running Ansible from Jenkins?**
**A:** Jenkins is a CI tool, not an automation platform. AWX provides features Jenkins lacks: Inventory sync (auto-fetch hosts from AWS), granular RBAC for specific tasks (not just "run pipeline"), and "Surveys" (user-friendly forms to input variables before running a job).

**Q: How does AWX handle dynamic inventory?**
**A:** AWX has built-in support for syncing inventory from cloud providers. You configure "Inventory Sources" (e.g., EC2 source), and AWX runs the sync periodically to update the database cache of hosts.
