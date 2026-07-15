### Intro (01)
- `what-is-iac / what-is-terraform / usecases-and-benefits` — Declarative vs imperative. HCL, providers, statefile. `init→plan→apply`. Multi-cloud, CI/CD, drift, auto-scaling.
- `cac-vs-ioc / installing-terraform` — IoC provisions (TF), CaC configures (Ansible). Combined pipeline. apt/yum/brew/binary/tfenv.

### HCL (02) — `what-is-hcl / basic-syntax`
- Declarative, JSON-compat. Blocks, arguments, for-expressions, dynamic blocks, splat `[*]`. Types: string/num/bool/list/map/set/object/tuple. Heredoc `<<-EOT`.

### Providers (03)
- `01-terraform-registry.md` — Public registry. Module source: `namespace/name/provider`. Private via HCP.
- `02-configuring-providers.md` — `alias` multi-region. Creds via env vars, never hardcode. `terraform.lock.hcl`.
- `03-versions.md` — `required_providers { version = "~> 5.0" }`. Pessimistic `~>`. `init -upgrade`.

### Resources (05)
- `01-resource-behavior.md` / `02-resource-lifecycle.md` — Arguments vs attributes. Implicit deps. `+` create, `~` in-place, `-/+` replace, `-` destroy. ForceNew triggers replacement (AMI change).
- `depends-on.md` / `count.md` / `for-each.md` — Explicit deps, indexed/stable creation. `count = N` (index-shifting risk), `for_each = map` (stable keys, preferred).
- `provider.md` / `lifecycle.md` — Aliased provider targeting. `create_before_destroy`, `prevent_destroy`, `ignore_changes`, `replace_triggered_by`, pre/postcondition.

### Variables (06)
- `01-input-variables.md` — Priority: CLI `-var` > `*.auto.tfvars` > `terraform.tfvars` > `TF_VAR_*` > default.
- `02-type-constraints.md` / `06-validation-rules.md` — `string/number/bool/list/map/set/object/tuple`. `validation { condition + error_message }` — fail-fast at plan.
- `03-variable-definition-file.md` — `.auto.tfvars` auto-loaded. JSON `.tfvars.json` supported. Never commit secrets.
- `04-local-values.md` — Computed reuse, not persisted. `merge()` tag composition.
- `05-environment-variables.md` — `TF_VAR_name`, `TF_LOG`, `TF_IN_AUTOMATION`.

### Outputs (07)
- `01-output-syntax.md` / `02-sensitive-outputs.md` — `terraform output -json/-raw`. `sensitive = true` hides CLI, NOT state. Propagation enforced.
- `03-preconditions.md` — Validates output post-creation before consumer receives value.

### Format & Validate (08)
- `01-terraform-fmt.md` — Canonical 2-space. `-check -recursive -diff` in CI.
- `02-terraform-validate.md` — Static HCL syntax + provider schema. Needs `init`.
- `03-tflint.md` — Pluggable linter. Provider-specific rules. Custom Go rules via SDK.

### State (09) — `sensitive / versioning / splitting / import / locking / remote / best-practices`
- `sensitive-data` — State stores all attrs incl secrets. Remote + encryption + secret managers.
- `versioning / locking` — S3 versioning (enable on bucket). Locking: S3+DynamoDB, Consul. `force-unlock`.
- `splitting-state / import` — Dir-based. `state mv -state-out`. CLI `import`, v1.5+ `import {}` + `plan -generate-config-out`.
- `remote-state / best-practices` — Backends: S3/GCS/Azure/Consul/TFC. Encrypt, lock, env separation, no git.
- `08-inspect-modify-state/` — `graph` DOT/DAG, `show` single, `list` inventory, `output` read, `rm`/`mv`/`pull`/`push`/`replace-provider`/`force-unlock`.

### Deployment & Cleanup (10–11)
- `01-terraform-plan.md` / `02-terraform-apply.md` / `03-terraform-parallelism.md` — `-out=tfplan`, `-refresh-only`. `-auto-approve`, default parallelism=10.
- `01-terraform-destroy.md` — Destroys all managed. `-target` for selective.

### Modules (12) / Provisioners (13) / CI/CD (16)
- `modules/` root-vs-child, published, local, inputs-outputs, best-practices — Provider in root only. Version pin. Contract: vars in, outputs out.
- `provisioners/` file/local-exec/remote-exec/custom — Last resort. Prefer user_data/Ansible. `when = destroy`.
- `github-actions/circle-ci/gitlab-ci/jenkins` — Plan on PR, apply on merge. Orbs/state/tokens/approval.

### Scaling (18)
- `splitting-large-state / parallelism / deployment-workflow / version-management / terragrunt / infracost` — Layer-based, DAG parallelism, git-flow, tfenv, DRY configs, cost estimation.

### Testing (19)
- `unit / contract / integration / end-to-end / testing-modules` — Native `terraform test`, Terratest Go lib. Mock providers for isolation.

### Security (20)
- `secret-management / compliance-sentinel / terrascan / checkov / trivy / kics` — Vault/Secrets Manager, policy-as-code, SAST 500+ rules, CIS benchmarks, graph-based scanning.

### HCP (21)
- `what-and-when / enterprise / authentication / workspaces / vcs-integration / run-tasks` — TFC SaaS, Sentinel, OIDC, CLI vs VCS workspaces, webhook PR comments, post-plan hooks.
