---
---

## Summary
In AWX, a **Job Template** defines *how* to run a playbook (Which playbook? Which inventory? Which credentials?). A **Job** is a single execution of that template. A **Workflow** is a visual orchestrator that chains multiple Job Templates together, allowing logic like "Run A, if successful Run B, if failed Run C".

## Detailed Explanation

### Job Templates
A definition that combines:
*   **Project**: Git repo containing playbooks.
*   **Playbook**: The specific YAML file (e.g., `deploy.yml`).
*   **Inventory**: Target hosts.
*   **Credentials**: SSH keys/AWS keys.
*   **Variables**: Extra vars to inject.

### Surveys
"Surveys" allow you to create a user-friendly form (Questions) that pops up when a user launches a job.
*   *Question*: "Which version to deploy?" (Dropdown: v1.0, v1.1).
*   *Result*: Passes `version: v1.1` as a variable to the playbook.

### Workflows
Visual editor to connect templates.
*   **Parallelism**: Run "Update Web" and "Update DB" simultaneously.
*   **Conditionals**: "On Success" (Green line), "On Failure" (Red line), "Always" (Blue line).

## Go-Specific Context/Examples

You can use a Go webhook handler to trigger complex workflows. For example, a GitHub webhook hits your Go app -> Go app validates the request -> Go app triggers an AWX Workflow Job via API.

## Interview Questions

**Q: Why use a Workflow instead of a giant playbook with `import_playbook`?**
**A:**
1.  **Modularity**: Re-use small, specific job templates in multiple workflows.
2.  **Error Handling**: Better visual logic for "If A fails, do B".
3.  **Inventory Mixing**: Step A can target "AWS Inventory", Step B can target "VMware Inventory". A single playbook run is typically bound to one inventory.

**Q: What is a "Sliced Job"?**
**A:** Job Slicing allows you to split a large job (e.g., update 1000 hosts) into multiple smaller jobs (slices) that run in parallel across the AWX cluster's capacity, speeding up execution.

**Q: Can you schedule Jobs in AWX?**
**A:** Yes, AWX has a built-in scheduler (cron-like) to run Job Templates or Workflows at specific times or intervals (e.g., "Backup Database every night at 2 AM").
