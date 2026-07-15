### 01-introduction
- `01-what-is-ansible.md` — Agentless, YAML playbooks, SSH/WinRM, idempotent, declarative. 4 domains: config mgmt, deploy, orchestration, provisioning
- `02-architecture.md` — Control node → SSH → managed nodes (Python only). Modules pushed → temp dir → JSON. 3000+ modules. Forks for parallelism
- `03-push-vs-pull-model.md` — Push: on-demand via SSH. Pull: `ansible-pull` + Git + cron. Hybrid common: push bootstrap → pull maintenance
- `04-agentless-architecture.md` — No agent daemon. SSH pipelining. raw/script modules for bootstrapping. Connection plugins: ssh, winrm, docker, local

### 02-installation-and-configuration
- `01-installing-ansible.md` — `pip install ansible` (latest) vs `apt install` (stable). `ansible-core` (minimal) vs community `ansible`. Control node: Linux/macOS
- `02-configuration-ansible-cfg.md` — Precedence: `ANSIBLE_CONFIG` > `./` > `~/` > `/etc/`. Keys: `forks`, `pipelining`, `host_key_checking`
- `03-setting-up-inventory.md` — Static (INI/YAML) + Dynamic (executable/plugin). Groups, children, `host_vars/`, `group_vars/`. `-i` flag
- `04-inventory-basics.md` — `all` group. `--list-hosts`. localhost exception. `host_vars` > `group_vars`. Variables: inline, group_vars, host_vars

### 03-ad-hoc-commands
- `01-using-ad-hoc-commands.md` — `ansible <pattern> -m <module> -a "<args>"`. Default module: `command`. `-b` for become/sudo
- `02-common-modules-ping-shell-command.md` — ping (SSH+Python, not ICMP). command (no shell meta). shell (pipes, redirects, injection risk). raw (no Python)
- `03-managing-files-and-packages.md` — copy, file, fetch, template. apt/yum: `state=present` (safe) vs `latest` (risky). service for state control

### 04-playbooks
- `01-playbook-basics-yaml.md` — YAML: `---`, plays → hosts → tasks. 2-space indent. `--syntax-check`. Play ≠ Playbook
- `02-writing-your-first-playbook.md` — Green=ok, Yellow=changed, Red=failed. `--check` dry-run. `become: yes`. Idempotent by design
- `03-tasks-and-modules.md` — Task = module + args. register (capture output), ignore_errors, when, loop, notify
- `04-handlers-and-notifications.md` — Run once at end, deduplicated. `--force-handlers`. Chainable via notify. Fires only on change
- `05-variables-and-facts.md` — Precedence: extra vars > host_vars > group_vars > play vars > defaults. `gather_facts: no` for speed. Facts: `ansible_os_family`, etc.

### 05-control-structures
- `01-conditionals-when.md` — `when:` implicit Jinja2 (no `{{ }}`). AND=list, OR=`or`. `failed_when` for error classification. `is defined`
- `02-loops-with-items.md` — `loop:` (no flatten) vs `with_items` (auto-flatten). `loop_control.index_var`. register → `.results[]`. Pass list to `name:` for packages (faster)
- `03-blocks-and-error-handling.md` — block → rescue → always. rescue clears error on success. Prefer over `ignore_errors` for rollback
- `04-tags.md` — `--tags`, `--skip-tags`. Special: `always`, `never`, `untagged`. Tags inherited on blocks/roles

### 06-roles-and-content-reuse
- `01-introduction-to-roles.md` — Reusable packages. `import_role` (static) vs `include_role` (dynamic). Galaxy ≈ package registry
- `02-role-directory-structure.md` — tasks/, handlers/, defaults/, vars/, files/, templates/, meta/, tests/. `ansible-galaxy init`. defaults < vars
- `03-using-ansible-galaxy.md` — `requirements.yml` for pinning. `--force` to override cache. `ansible-galaxy collection install`. Audit community code
- `04-creating-custom-roles.md` — Prefix vars (`role_`). `meta/main.yml` for deps. `include_vars: "{{ ansible_os_family }}.yml"` for OS split
- `05-collections.md` — FQCN: `namespace.collection.module`. Decouples release cycles. `~/.ansible/collections`

### 07-advanced-topics
- `01-ansible-vault-security.md` — AES-256. `encrypt_string`, `--ask-vault-pass`. Vault IDs for multi-password. Never commit vault password
- `02-dynamic-inventory.md` — Plugins > Scripts. `_meta` avoids N+1 queries. Keyed groups for cloud tag targeting
- `03-optimization-and-speed.md` — pipelining, forks (50+), fact caching (JSON/Redis), strategy:free, third-party Mitogen (2-5×)
- `04-custom-modules.md` — Any language → args file → JSON stdout. Contract: `{"changed":bool,"failed":bool}`. Go: static binary
- `05-jinja2-templating.md` — `{{ }}` output, `{% %}` logic. Filters via `|`. Sandboxed, no arbitrary Python. Whitespace: `{%-`, `-%}`

### 08-ansible-tower-awx
- `01-introduction-to-awx-tower.md` — AWX (OSS upstream) / Tower (Red Hat). Web UI + REST API. RBAC, credential vault, inventory sync
- `02-dashboard-and-management.md` — Hierarchy: Org → Team → User. Per-object RBAC. `/api/v2/metrics/`
- `03-jobs-and-workflows.md` — Template (def) → Job (run). Workflows: conditional edges. Surveys, job slicing, cron scheduling

### 09-testing
- `01-ansible-lint.md` — Static: idempotency, FQCN, deprecated. `.ansible-lint` config. `noqa` per-task skip
- `02-molecule-testing.md` — Create → Converge → Idempotence (2nd run=0 changed) → Verify → Destroy. Docker, Vagrant, EC2 drivers
- `03-ci-cd-pipelines.md` — Lint → Test → Staging → Production. `--check` for PR dry-runs. SSH keys in CI secrets. GitOps pattern

### Go cross-cutting
- `os/exec`: `cmd.Dir` for ansible.cfg, `ANSIBLE_CONFIG` env. Dynamic inventory: Go `--list` → JSON `_meta`
- Custom module: compile → binary, JSON stdin/stdout. AWX: REST + Bearer. Vault: `sosedoff/ansible-vault-go`