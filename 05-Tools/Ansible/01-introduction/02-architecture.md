---
tags: ['ansible', 'automation', 'devops', 'tools', 'roadmap']
---

# Ansible Architecture

## Summary

Ansible's architecture consists of a control node (where Ansible runs), managed nodes (target systems), inventory (host definitions), playbooks (automation scripts), modules (units of work), and plugins (extend functionality). The control node connects to managed nodes via SSH/WinRM, pushes Python modules, executes them, and removes them after completion. This stateless, agentless design makes Ansible simple to deploy and maintain.

## Detailed Explanation

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        CONTROL NODE                              │
│  ┌──────────┐  ┌───────────┐  ┌──────────┐  ┌───────────────┐  │
│  │ Ansible  │  │ Inventory │  │Playbooks │  │   Modules/    │  │
│  │  Engine  │  │  (hosts)  │  │  (YAML)  │  │   Plugins     │  │
│  └────┬─────┘  └─────┬─────┘  └────┬─────┘  └───────┬───────┘  │
│       │              │             │                │          │
│       └──────────────┴─────────────┴────────────────┘          │
│                              │                                   │
│                        SSH / WinRM                               │
└──────────────────────────────┬──────────────────────────────────┘
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
        ▼                      ▼                      ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│  MANAGED NODE │    │  MANAGED NODE │    │  MANAGED NODE │
│   (web-01)    │    │   (web-02)    │    │   (db-01)     │
│   Python      │    │   Python      │    │   Python      │
└───────────────┘    └───────────────┘    └───────────────┘
```

### Control Node

```bash
# The machine where Ansible is installed and run from
# Requirements:
# - Linux, macOS, or WSL (not native Windows)
# - Python 3.9+ 
# - SSH client

# Control node components:
# 1. Ansible binaries (ansible, ansible-playbook, etc.)
# 2. Configuration files (ansible.cfg)
# 3. Inventory files
# 4. Playbooks and roles
# 5. SSH keys for authentication

# Check Ansible installation
ansible --version
# ansible [core 2.15.0]
#   config file = /etc/ansible/ansible.cfg
#   python version = 3.11.0
#   jinja version = 3.1.2
#   libyaml = True
```

### Managed Nodes

```bash
# Target systems that Ansible manages
# Requirements:
# - SSH server (Linux/Unix) or WinRM (Windows)
# - Python 2.7+ or 3.5+ (for most modules)
# - No Ansible software installation needed

# Ansible's execution process on managed nodes:
# 1. Connects via SSH
# 2. Creates temp directory
# 3. Transfers Python module code
# 4. Executes module with Python
# 5. Captures output (JSON)
# 6. Removes temp files
# 7. Returns results to control node

# Verify connectivity
ansible all -m ping
# web-01 | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
```

### Inventory

```ini
# Static inventory file - defines managed hosts
# /etc/ansible/hosts or custom path

# Simple format
web-01
web-02
db-01

# With groups
[webservers]
web-01 ansible_host=192.168.1.10
web-02 ansible_host=192.168.1.11

[databases]
db-01 ansible_host=192.168.1.20

[production:children]
webservers
databases

# Group variables
[webservers:vars]
ansible_user=deploy
http_port=80

# Host variables
web-01 ansible_port=2222 custom_var=value
```

```yaml
# YAML inventory format
all:
  children:
    webservers:
      hosts:
        web-01:
          ansible_host: 192.168.1.10
        web-02:
          ansible_host: 192.168.1.11
      vars:
        http_port: 80
    
    databases:
      hosts:
        db-01:
          ansible_host: 192.168.1.20
          mysql_port: 3306
    
    production:
      children:
        webservers:
        databases:
```

### Modules

```yaml
# Modules are units of work - each performs a specific task
# Ansible includes 3000+ built-in modules

# Module categories:
# - System: user, group, service, cron, timezone
# - Files: file, copy, template, lineinfile, fetch
# - Packaging: apt, yum, dnf, pip, npm
# - Cloud: ec2, azure, gcp modules
# - Database: mysql_db, postgresql_db
# - Network: ios_config, nxos_command

# Using modules in tasks
- name: Manage system user
  ansible.builtin.user:
    name: deploy
    groups: sudo
    state: present

- name: Install packages
  ansible.builtin.apt:
    name:
      - nginx
      - python3
    state: present
    update_cache: yes

- name: Copy configuration file
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    mode: '0644'
```

### Plugins

```yaml
# Plugins extend Ansible's core functionality
# Types of plugins:

# 1. CONNECTION PLUGINS - How to connect to hosts
# - ssh (default)
# - winrm (Windows)
# - docker
# - local

# 2. LOOKUP PLUGINS - Retrieve data from external sources
- name: Read secret from file
  debug:
    msg: "{{ lookup('file', '/path/to/secret') }}"

- name: Get environment variable
  debug:
    msg: "{{ lookup('env', 'HOME') }}"

# 3. FILTER PLUGINS - Transform data
- debug:
    msg: "{{ my_list | join(', ') }}"

- debug:
    msg: "{{ my_string | upper }}"

# 4. CALLBACK PLUGINS - Customize output
# Configure in ansible.cfg:
# [defaults]
# stdout_callback = yaml

# 5. INVENTORY PLUGINS - Dynamic inventory sources
# - aws_ec2
# - azure_rm
# - gcp_compute
```

### Playbooks and Plays

```yaml
# Playbook: YAML file containing one or more plays
# Play: Maps hosts to tasks

---
# This is a playbook with two plays
- name: Configure web servers
  hosts: webservers
  become: yes
  gather_facts: yes
  
  vars:
    http_port: 80
  
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
    
    - name: Configure nginx
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx
  
  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted

- name: Configure database servers
  hosts: databases
  become: yes
  
  tasks:
    - name: Install MySQL
      apt:
        name: mysql-server
        state: present
```

### Execution Flow

```bash
# 1. Parse inventory to identify target hosts
# 2. Load playbook and validate YAML
# 3. For each play:
#    a. Gather facts from hosts (if enabled)
#    b. Execute tasks in order
#    c. Run handlers if notified
# 4. Report results

# Execution order within a play:
# 1. pre_tasks
# 2. roles (in order listed)
# 3. tasks
# 4. post_tasks
# 5. handlers (triggered by notify)

# Parallel execution
# Ansible runs tasks on multiple hosts simultaneously
# Default: 5 hosts at a time (forks)

# Configure parallelism
ansible-playbook site.yml --forks=20
# Or in ansible.cfg:
# [defaults]
# forks = 20
```

### Facts and Variables

```yaml
# FACTS: Auto-collected system information
# Gathered at play start (gather_facts: yes)

- name: Show facts
  hosts: all
  tasks:
    - debug:
        msg: |
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          IP: {{ ansible_default_ipv4.address }}
          Memory: {{ ansible_memtotal_mb }}MB
          CPU: {{ ansible_processor_cores }} cores

# VARIABLES: User-defined data
# Sources (precedence low to high):
# 1. Role defaults
# 2. Inventory vars
# 3. Playbook vars
# 4. Role vars
# 5. Extra vars (-e)

- name: Use variables
  hosts: webservers
  vars:
    app_name: myapp
    app_port: 8080
  
  tasks:
    - debug:
        msg: "Deploying {{ app_name }} on port {{ app_port }}"
```

## Interview Questions

### Q1: What is the difference between a control node and managed node?
**A:** The control node is where Ansible is installed and playbooks are executed from. Managed nodes are the target systems being configured. The control node pushes configurations to managed nodes via SSH/WinRM - managed nodes don't need Ansible installed.

### Q2: What are the minimum requirements for a managed node?
**A:** For Linux: SSH server access and Python 2.7+/3.5+. For Windows: WinRM enabled and PowerShell 3.0+. No Ansible software installation is required on managed nodes.

### Q3: What is an Ansible module?
**A:** A module is a unit of code that performs a specific task (install package, copy file, manage user). Modules are idempotent and return JSON results. Ansible ships with 3000+ built-in modules and you can write custom ones in Python.

### Q4: How does Ansible execute tasks on remote hosts?
**A:** Ansible connects via SSH, transfers Python module code to a temp directory on the remote host, executes the module using the host's Python interpreter, captures the JSON output, cleans up temp files, and returns results to the control node.

### Q5: What is the difference between plugins and modules?
**A:** Modules execute on managed nodes to perform tasks (install packages, manage files). Plugins run on the control node to extend Ansible functionality - connection plugins, lookup plugins for data retrieval, filter plugins for data transformation, callback plugins for output formatting.

### Q6: What is an Ansible playbook and what is a play?
**A:** A playbook is a YAML file containing one or more plays. A play maps a set of hosts to a list of tasks. Each play specifies which hosts to target, what variables to use, and what tasks to execute.

### Q7: What are Ansible facts?
**A:** Facts are system information automatically collected from managed nodes when `gather_facts: yes`. They include OS details, network configuration, hardware info, etc. Facts are stored as variables (e.g., `ansible_distribution`, `ansible_memtotal_mb`).

### Q8: How does Ansible handle parallel execution?
**A:** Ansible executes tasks on multiple hosts simultaneously using "forks." The default is 5 parallel connections. This can be configured with `--forks` flag or `forks` setting in `ansible.cfg`. Tasks are executed in order per host, but hosts are processed in parallel.
