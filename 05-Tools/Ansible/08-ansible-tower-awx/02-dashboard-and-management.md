---
---

## Summary
The AWX Dashboard provides a high-level visual overview of your automation infrastructure. It displays the status of recent job runs (Success/Failed), the total number of managed hosts (and their status), and synchronization health. It is the entry point for managing Organizations, Teams, and Users.

## Detailed Explanation

### Dashboard Views
1.  **Job Status Graph**: A time-series chart showing successful vs. failed jobs over time. Helps spot trends (e.g., "Why do deployments fail every Friday?").
2.  **Recent Templates**: Quick access to frequently run playbooks.
3.  **Host Status**: Count of hosts reachable vs unreachable.

### Management Hierarchy
1.  **Organization**: The top-level container (e.g., "Engineering", "Finance"). Inventories and Projects belong to an Org.
2.  **Team**: A group of users within an Org (e.g., "DevOps", "Frontend").
3.  **User**: An individual account.
4.  **RBAC**: Permissions are assigned to Teams/Users on specific objects (e.g., "DevOps team has **Execute** access on 'Deploy App' Job Template").

## Go-Specific Context/Examples

You can build a custom "Status Board" in Go (using something like `termui` or a web server) that pulls metrics from the AWX API to display on a TV in the office.

### Endpoint: `/api/v2/metrics/`
AWX exposes Prometheus-style metrics if configured, or you can query `/api/v2/dashboard/` for JSON summaries.

## Interview Questions

**Q: Can a User belong to multiple Organizations?**
**A:** Yes. A user can be a member of multiple organizations and multiple teams, with different permission levels in each.

**Q: What happens if a job fails in the dashboard?**
**A:** It shows as red (Failed). You can click into the Job ID to see the standard output (stdout) of the ansible-playbook run, exactly as it would appear in the terminal, to debug the error.

**Q: How do you segregate production vs staging access?**
**A:** Create two Organizations (or two Inventories). Create a "Read-Only" team and an "Admin" team. Give "Admin" permission to the Staging Inventory, but only "Read" (or no access) to the Production Inventory.
