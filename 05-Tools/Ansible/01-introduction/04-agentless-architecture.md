---
tags: ['ansible', 'automation', 'devops', 'tools', 'roadmap']
---

# Agentless Architecture

## Summary

Ansible's agentless architecture means no special software (daemon or agent) needs to be installed and maintained on managed nodes. Ansible connects via standard SSH (Linux/Unix) or WinRM (Windows), transfers temporary Python modules, executes them, and removes them after completion. This simplifies deployment, reduces security surface, eliminates version compatibility issues, and makes Ansible quick to adopt with minimal infrastructure changes.

## Detailed Explanation

### How Agentless Works

```
┌──────────────────────────────────────────────────────────────┐
│                      CONTROL NODE                             │
│  ansible-playbook site.yml                                    │
│                          │                                    │
│  1. Parse playbook      │                                    │
│  2. Build module code    │                                    │
│  3. Connect via SSH      │                                    │
└──────────────────────────┼───────────────────────────────────┘
                           │
                      SSH/WinRM
                           │
                           ▼
┌──────────────────────────────────────────────────────────────┐
│                      MANAGED NODE                             │
│                                                               │
│  4. Create temp directory (~/.ansible/tmp/...)               │
│  5. Transfer module (Python code)                            │
│  6. Execute module with Python interpreter                   │
│  7. Module makes changes and returns JSON result             │
│  8. Delete temp directory                                    │
│  9. Return result to control node                            │
│                                                               │
│  REQUIREMENTS:                                                │
│  - SSH server (sshd)                                         │
│  - Python 2.7+ or 3.5+                                       │
│  - User with appropriate permissions                         │
└──────────────────────────────────────────────────────────────┘
```

### Managed Node Requirements

```bash
# LINUX/UNIX REQUIREMENTS:
# 1. SSH server running and accessible
systemctl status sshd

# 2. Python installed (most distros have it)
python3 --version  # or python --version

# 3. User account with SSH access
# 4. sudo if become: yes is needed

# WINDOWS REQUIREMENTS:
# 1. PowerShell 3.0+
$PSVersionTable.PSVersion

# 2. WinRM enabled and configured
winrm quickconfig

# 3. .NET Framework 4.0+

# VERIFY CONNECTIVITY:
ansible all -m ping
# If successful, managed nodes are ready

# MINIMAL PYTHON MODULES:
# raw and script modules don't require Python
# Used for bootstrapping nodes without Python
ansible webserver -m raw -a "apt-get install -y python3"
```

### SSH Connection Details

```bash
# Ansible uses SSH multiplexing by default
# ControlPersist keeps connections open for reuse

# Default SSH configuration in ansible.cfg:
[ssh_connection]
ssh_args = -C -o ControlMaster=auto -o ControlPersist=60s
pipelining = True

# Connection variables per host:
# ansible_host - IP/hostname to connect to
# ansible_port - SSH port (default 22)
# ansible_user - SSH username
# ansible_ssh_private_key_file - Path to SSH key
# ansible_ssh_common_args - Additional SSH arguments

# Inventory example with connection vars
[webservers]
web-01 ansible_host=192.168.1.10 ansible_user=deploy
web-02 ansible_host=192.168.1.11 ansible_port=2222

# Use bastion/jump host
[webservers:vars]
ansible_ssh_common_args='-o ProxyJump=bastion.example.com'
```

```yaml
# SSH connection in playbook
---
- name: Configure via SSH
  hosts: webservers
  
  vars:
    ansible_user: deploy
    ansible_ssh_private_key_file: ~/.ssh/deploy_key
    ansible_become: yes
    ansible_become_method: sudo
  
  tasks:
    - name: Verify connection
      ping:
```

### Windows WinRM Connection

```powershell
# Configure WinRM on Windows (run as Administrator)

# Option 1: Basic setup
winrm quickconfig

# Option 2: Enable for Ansible (more permissive)
# Download and run ConfigureRemotingForAnsible.ps1
$url = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"
(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)
powershell.exe -ExecutionPolicy ByPass -File $file

# Verify WinRM listener
winrm enumerate winrm/config/Listener
```

```yaml
# Windows connection in inventory
[windows]
win-01 ansible_host=192.168.1.50

[windows:vars]
ansible_user=Administrator
ansible_password="{{ vault_win_password }}"
ansible_connection=winrm
ansible_winrm_transport=ntlm
ansible_winrm_server_cert_validation=ignore
ansible_port=5986  # HTTPS
```

### Pipelining

```ini
# Pipelining reduces SSH operations
# Instead of: copy module -> execute -> delete
# Does: pipe module directly to Python stdin

# Enable in ansible.cfg
[ssh_connection]
pipelining = True

# Requirements for pipelining:
# - requiretty must be disabled in sudoers
# - Or don't use become

# Disable requiretty on managed nodes:
# visudo
# Comment out: Defaults    requiretty
```

### Connection Plugins

```yaml
# Different connection types for different scenarios

# SSH (default for Linux/Unix)
ansible_connection: ssh

# WinRM (for Windows)
ansible_connection: winrm

# Local (run on control node itself)
ansible_connection: local

# Docker (connect to containers)
ansible_connection: docker

# Network devices
ansible_connection: network_cli
ansible_network_os: ios

# Example inventory with different connections
---
all:
  children:
    linux:
      hosts:
        web-01:
          ansible_connection: ssh
    
    windows:
      hosts:
        win-01:
          ansible_connection: winrm
    
    containers:
      hosts:
        app-container:
          ansible_connection: docker
    
    localhost:
      hosts:
        local:
          ansible_connection: local
```

### Agentless vs Agent-Based

```yaml
# AGENTLESS (Ansible)
advantages:
  - No software to install on managed nodes
  - No agent version compatibility issues  
  - No agent daemon consuming resources
  - Smaller security attack surface
  - Works immediately on any SSH-accessible system
  - Simpler infrastructure
  
disadvantages:
  - Requires SSH/WinRM access
  - Slightly slower (module transfer overhead)
  - No persistent local state
  - No automatic drift correction
  - Requires Python on managed nodes

# AGENT-BASED (Puppet, Chef)
advantages:
  - Can work behind firewalls (outbound only)
  - Persistent daemon for pull model
  - Local state caching
  - Built-in drift detection
  - Can be faster (pre-installed code)
  
disadvantages:
  - Agent installation and maintenance
  - Agent version compatibility
  - Resource consumption (always running)
  - Larger attack surface
  - More complex initial setup
```

### Raw and Script Modules

```yaml
# For nodes without Python or for bootstrapping

# RAW MODULE - Execute command without Python
- name: Install Python on minimal system
  raw: apt-get update && apt-get install -y python3
  
- name: Check if Python exists
  raw: which python3
  register: python_check
  failed_when: false

# SCRIPT MODULE - Run local script on remote
- name: Run custom script
  script: /local/path/setup.sh
  args:
    creates: /etc/myapp.conf  # Idempotency check

# Bootstrap pattern
---
- name: Bootstrap managed nodes
  hosts: new_servers
  gather_facts: no  # Facts require Python
  
  tasks:
    - name: Install Python
      raw: |
        test -e /usr/bin/python3 || \
        (apt-get update && apt-get install -y python3)
      changed_when: false
    
    - name: Now gather facts
      setup:
```

### Security Implications

```bash
# REDUCED ATTACK SURFACE
# No listening daemon on managed nodes
# No open ports specific to configuration management
# Only standard SSH port exposed

# CREDENTIAL MANAGEMENT
# SSH keys on control node
# Use ssh-agent for key management
eval $(ssh-agent)
ssh-add ~/.ssh/ansible_key

# PRIVILEGE ESCALATION
# become/sudo only when needed
# ansible_become_password can be vaulted

# BEST PRACTICES:
# 1. Use dedicated service account (not root)
# 2. Limit sudo permissions via sudoers
# 3. Use SSH key authentication (not passwords)
# 4. Rotate keys regularly
# 5. Use Ansible Vault for secrets
# 6. Audit SSH access logs
```

```yaml
# Restricted sudoers for Ansible user
# /etc/sudoers.d/ansible
deploy ALL=(ALL) NOPASSWD: /usr/bin/apt-get, /bin/systemctl, /usr/bin/tee

# Playbook with minimal privilege
---
- name: Secure deployment
  hosts: webservers
  become: yes
  become_user: deploy  # Not root where possible
  become_method: sudo
  
  tasks:
    - name: Task needing root
      apt:
        name: nginx
        state: present
```

## Interview Questions

### Q1: What does "agentless" mean in Ansible's context?
**A:** No special software needs to be installed on managed nodes. Ansible uses existing SSH (Linux) or WinRM (Windows) protocols, transfers Python modules temporarily, executes them, and removes them. Only Python and SSH/WinRM are required.

### Q2: What are the minimum requirements for an Ansible managed node?
**A:** Linux/Unix: SSH server and Python 2.7+/3.5+. Windows: WinRM enabled, PowerShell 3.0+, and .NET Framework 4.0+. A user account with appropriate permissions is needed for both.

### Q3: How does Ansible execute commands on remote hosts without an agent?
**A:** Ansible: 1) Connects via SSH, 2) Creates a temp directory, 3) Transfers Python module code, 4) Executes module using the host's Python interpreter, 5) Captures JSON output, 6) Removes temp files, 7) Returns results to control node.

### Q4: What is SSH pipelining and why is it important?
**A:** Pipelining reduces SSH operations by piping module code directly to Python's stdin instead of copying files to disk first. This significantly speeds up execution but requires `requiretty` to be disabled in sudoers.

### Q5: How do you manage Windows systems with Ansible?
**A:** Use WinRM (Windows Remote Management) instead of SSH. Configure WinRM on Windows hosts, set `ansible_connection: winrm` in inventory, and use Windows-specific modules. Authentication can be NTLM, Kerberos, or CredSSP.

### Q6: What are the security advantages of agentless architecture?
**A:** Smaller attack surface (no listening daemons), no agent software to keep patched, uses standard secure protocols (SSH), no additional ports to open, and credentials stay on control node rather than being distributed.

### Q7: How can you configure a node that doesn't have Python installed?
**A:** Use the `raw` module which doesn't require Python - it executes commands directly via SSH. First install Python with raw (`raw: apt-get install python3`), then proceed with normal modules.

### Q8: What connection plugins does Ansible support besides SSH?
**A:** `winrm` (Windows), `local` (control node itself), `docker` (containers), `network_cli` (network devices), `paramiko_ssh` (pure Python SSH), `psrp` (PowerShell Remoting Protocol), and custom plugins for specialized environments.
