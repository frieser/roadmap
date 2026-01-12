---
tags: ['ansible', 'automation', 'devops', 'tools', 'roadmap']
---

# Push vs Pull Model

## Summary

Configuration management tools use either push or pull models for delivering configurations. In the **push model** (Ansible's default), the control node initiates connections and pushes configurations to managed nodes on-demand. In the **pull model** (Puppet, Chef default), agents on managed nodes periodically poll a central server for updates. Ansible primarily uses push but supports pull via `ansible-pull`. Each model has trade-offs regarding scalability, control, and infrastructure requirements.

## Detailed Explanation

### Push Model (Ansible Default)

```
┌─────────────────┐
│  CONTROL NODE   │
│    (Ansible)    │
└────────┬────────┘
         │
         │  SSH push (on-demand)
         │
    ┌────┴────┬────────┬────────┐
    │         │        │        │
    ▼         ▼        ▼        ▼
┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐
│Host 1 │ │Host 2 │ │Host 3 │ │Host N │
│(no    │ │(no    │ │(no    │ │(no    │
│agent) │ │agent) │ │agent) │ │agent) │
└───────┘ └───────┘ └───────┘ └───────┘
```

```bash
# Push model execution
# You run ansible-playbook, it pushes to hosts immediately

# Run playbook (push)
ansible-playbook site.yml

# Ad-hoc command (push)
ansible all -m apt -a "name=nginx state=present"

# Characteristics of PUSH:
# ✓ Immediate execution - run when you need
# ✓ No infrastructure overhead - no pull server needed
# ✓ Administrator control - you decide when to push
# ✓ Simple network requirements - outbound SSH only
# ✓ Visible execution - see results in real-time

# Limitations:
# ✗ Requires network access at push time
# ✗ Firewalls may block incoming connections to hosts
# ✗ No automatic drift correction
# ✗ Scalability requires multiple control nodes
```

### Pull Model (ansible-pull)

```
┌─────────────────────────────────────────────────────────┐
│                     GIT REPOSITORY                       │
│              (playbooks, roles, inventory)               │
└────────────────────────┬────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
   ┌─────────┐      ┌─────────┐      ┌─────────┐
   │ Host 1  │      │ Host 2  │      │ Host 3  │
   │ansible- │      │ansible- │      │ansible- │
   │  pull   │      │  pull   │      │  pull   │
   │ (cron)  │      │ (cron)  │      │ (cron)  │
   └─────────┘      └─────────┘      └─────────┘
   Pulls from repo every N minutes
```

```bash
# ansible-pull runs on each managed node
# Pulls playbooks from Git and runs locally

# Basic ansible-pull command
ansible-pull -U https://github.com/org/ansible-config.git

# With specific playbook
ansible-pull -U https://github.com/org/ansible-config.git local.yml

# With inventory
ansible-pull -U https://github.com/org/ansible-config.git \
  -i inventory/hosts \
  -e "target_host=$(hostname)"

# Set up cron job for periodic pull
ansible-pull -U https://github.com/org/ansible-config.git \
  --only-if-changed \
  -o \
  -C main \
  local.yml
```

```yaml
# Setup ansible-pull via playbook (bootstrap)
---
- name: Configure ansible-pull
  hosts: all
  become: yes
  
  tasks:
    - name: Install Ansible
      apt:
        name: ansible
        state: present
    
    - name: Create ansible-pull cron job
      cron:
        name: "ansible-pull"
        minute: "*/15"
        job: >
          ansible-pull 
          -U https://github.com/org/ansible-config.git 
          -o 
          --only-if-changed
          local.yml
          >> /var/log/ansible-pull.log 2>&1
        user: root
    
    - name: Setup log rotation
      copy:
        dest: /etc/logrotate.d/ansible-pull
        content: |
          /var/log/ansible-pull.log {
            weekly
            rotate 4
            compress
            missingok
          }
```

### Comparison

```yaml
# PUSH MODEL
advantages:
  - Immediate execution on demand
  - Real-time feedback and output
  - No agent installation required
  - Administrator-controlled timing
  - Simple to debug (interactive)
  - Lower infrastructure requirements
  
disadvantages:
  - Requires SSH access from control node
  - Firewall rules may be complex
  - Manual or scheduled execution needed
  - No automatic drift correction
  - Control node becomes bottleneck at scale

use_cases:
  - Ad-hoc changes and troubleshooting
  - CI/CD pipeline deployments
  - Orchestrated multi-tier deployments
  - Initial server provisioning
  - One-time configurations

# PULL MODEL
advantages:
  - Nodes only need outbound access (to Git)
  - Auto-correct configuration drift
  - Scales well (no central bottleneck)
  - Works behind firewalls/NAT
  - Self-healing systems
  
disadvantages:
  - Delayed execution (poll interval)
  - Requires Ansible on each node
  - Git repository is single point of failure
  - Less visibility into execution
  - More complex initial setup

use_cases:
  - Auto-scaling environments
  - Systems behind restrictive firewalls
  - Continuous compliance enforcement
  - Large distributed systems
  - Edge computing nodes
```

### Hybrid Approach

```yaml
# Many organizations use BOTH models

# 1. INITIAL PROVISIONING (Push)
# New server bootstrapped with push to install ansible-pull
---
- name: Bootstrap new server for pull mode
  hosts: new_servers
  become: yes
  
  tasks:
    - name: Install git and ansible
      apt:
        name:
          - git
          - ansible
        state: present
    
    - name: Configure ansible-pull cron
      cron:
        name: ansible-pull
        minute: "*/30"
        job: "ansible-pull -U {{ repo_url }} site.yml"
    
    - name: Run initial pull immediately
      command: ansible-pull -U {{ repo_url }} site.yml

# 2. ONGOING MAINTENANCE (Pull)
# Nodes self-update from Git repository

# 3. URGENT CHANGES (Push)
# Critical patches pushed immediately
ansible-playbook emergency-patch.yml
```

### AWX/Tower Scheduling

```yaml
# AWX/Tower provides push-like scheduling
# without ansible-pull complexity

# Scheduled jobs in AWX:
# - Run playbooks on schedule (like pull behavior)
# - Maintain visibility and logging
# - No ansible installation on managed nodes
# - Central management and RBAC

# Configuration:
# 1. Create Job Template
# 2. Add Schedule (cron-style)
# 3. Jobs run automatically at scheduled times
# 4. Results visible in AWX dashboard

# This gives "pull-like" benefits with push architecture:
# ✓ Regular enforcement of configuration
# ✓ No agent on managed nodes
# ✓ Central visibility
# ✓ Easy to modify schedules
```

### Security Considerations

```bash
# PUSH MODEL SECURITY:
# - SSH keys on control node are high-value targets
# - Control node needs network access to all hosts
# - Use bastion/jump hosts for segmented networks
# - Implement least-privilege SSH access

# PULL MODEL SECURITY:
# - Git repository credentials on each host
# - Use deploy keys with read-only access
# - Consider private Git server
# - Protect ansible-pull logs (may contain secrets)

# BOTH MODELS:
# - Use Ansible Vault for secrets
# - Implement proper key rotation
# - Monitor and audit execution logs
```

### Scalability Patterns

```bash
# PUSH SCALING:
# Problem: Control node bottleneck with 1000s of hosts

# Solution 1: Increase forks
ansible-playbook site.yml --forks=50

# Solution 2: Multiple control nodes
# Geographic distribution of control nodes
# Each manages subset of infrastructure

# Solution 3: AWX/Tower execution nodes
# Distribute work across multiple execution nodes

# PULL SCALING:
# Naturally scales - each node handles itself
# Git server is the only bottleneck
# Use Git mirrors for geographic distribution
# Consider artifact caching (Ansible collections)
```

## Interview Questions

### Q1: What is the difference between push and pull configuration management models?
**A:** Push model: control node initiates connection and pushes configuration to managed nodes on-demand (Ansible default). Pull model: managed nodes periodically poll a central server/repository for configuration updates (Puppet/Chef default). Push is immediate but requires connectivity; pull is autonomous but has delay.

### Q2: What is ansible-pull and when would you use it?
**A:** `ansible-pull` runs Ansible on the managed node itself, pulling playbooks from a Git repository. Use it for: auto-scaling environments where nodes self-configure, systems behind firewalls that can't accept incoming connections, or continuous drift correction without central scheduling.

### Q3: What are the advantages of push over pull?
**A:** Push advantages: immediate execution with real-time feedback, no agent installation, simpler debugging, lower infrastructure requirements, and precise control over execution timing. You know exactly when and what ran.

### Q4: Why might you choose pull model over push?
**A:** Pull model works when: nodes are behind restrictive firewalls (only outbound allowed), you need automatic drift correction, you have massive scale where central control nodes become bottlenecks, or nodes may be offline when you want to push changes.

### Q5: How would you implement ansible-pull in an environment?
**A:** Bootstrap each node with a push playbook that: installs Ansible, sets up Git access (deploy keys), creates a cron job running `ansible-pull -U <repo> --only-if-changed`, and configures logging. The cron job then keeps the node configured going forward.

### Q6: Can you use both push and pull together?
**A:** Yes, this is common. Use push for: initial provisioning, urgent patches, orchestrated deployments. Use pull for: routine configuration enforcement, auto-scaling instances, drift correction. The models are complementary.

### Q7: What is the main scalability challenge with the push model?
**A:** The control node becomes a bottleneck - it must maintain SSH connections to all hosts and has limited parallelism (forks). Solutions include: increasing forks, using multiple control nodes, using AWX/Tower with execution nodes, or implementing pull for large-scale infrastructure.

### Q8: How does drift detection differ between push and pull models?
**A:** Push model has no automatic drift detection - configuration only enforced when playbooks are run. Pull model continuously enforces configuration at poll intervals, automatically correcting drift. To get drift detection with push, you need scheduled job execution via AWX/Tower or external schedulers.
