### Platforms
- `00-version-control-systems.md` — DVCS, snapshots, full local copy
- `01-github.md` — cloud PRs/Issues/Actions | `02-gitlab.md` — self-hosted CI/CD | `03-bitbucket.md` — Atlassian stack

### `01-learn-the-basics/` — VCS concept, why use, Git vs SVN (DVCS), installing locally

### `02-what-is-a-repository/`
- `01-git-init.md` `.git/` | `01-local-vs-global-config.md` System>Global>Local
- `03-repository-initialization.md` init+ignore+README | `04-working-directory.md` 3 Trees
- `05-staging-area.md` Index | `06-committing-changes.md` SHA-1 snapshot | `07-intro-and-git-commands.md` `status→add→commit`

### `03-basic-collaboration/`
- clone, remote add/remove, push/pull, `.gitignore`, fetch (no merge), `git log`

### `04-branching-basics/`
- create, rename (`-m`), delete (`-d`/`push -d`), checkout/switch, merge

### `05-github-essentials/` — account, interface, profile README, public vs private

### `06-merge-strategies/`
- FF vs `--no-ff` | rebase (replay on base) | squash (N→1) | conflicts | `cherry-pick <sha>`

### `07-collaboration-on-github/`
- Fork (server) vs clone (local) | Issues | PR from fork | collaborators | PR template
- Labels, saved replies, `@mentions`, reactions, GFM comments, Discussions

### `08-best-practices/` — Conventional Commits, kebab-case branches, CONTRIBUTING.md, clean linear history
### `09-documentation/` — GFM, README.md, Wikis (separate repo), CITATION.cff

### `11-intermediate-git-topics/`
- Linear vs non-linear | HEAD / detached HEAD | `git log` options (`--oneline`/`--graph`)
- `revert` (safe inverse commit) | `reset --soft` (keep staged) | `--hard` (destructive wipe) | `--mixed` (default, unstage)

### `12-working-in-a-team/` — Projects (table/board), Kanban (Todo→Done), Roadmaps (Gantt), automations, collaborators vs members, orgs, `@org/team`

### `13-viewing-diffs/` — between commits, between branches, `--staged`, unstaged (default)
### `14-rewriting-history/` — `--amend`, interactive rebase, `filter-branch`, `push --force-with-lease`
### `15-submodules/` — repo-inside-repo, `add`/`update --init`

### `16-git-hooks/`
- Client vs server | usecases (lint/test/scan) | `commit-msg`, `post-checkout`, `post-update`, `pre-commit`, `pre-push`

### `17-tagging/` — lightweight/annotated, push `--tags`, checkout (detached HEAD), GitHub Releases = tag+binaries
### `18-github-workflow/` — `gh` CLI: auth, repo, issue, PR mgmt, `--json`+`jq` for scripting

### `19-github-actions/`
- `.yml` in `.github/workflows/` | triggers: `push`/`pr`/`schedule` (cron) | runners (hosted/self)
- `${{ github.ref/sha/actor }}` | `${{ secrets.X }}` | cache, artifacts, badges, marketplace

### `20-advanced-git-topics/`
- `01-git-reflog.md` safety net | `02-git-bisect.md` binary search bug | `03-git-worktree.md` multi-branch
- `04-git-attributes.md` per-path settings | `05-git-lfs.md` binary→text pointers

### `21-github-api/` — REST (HTTP, pagination), GraphQL (exact fields)
### `22-github-developer-tools/` — OAuth Apps vs GitHub Apps, registration (permissions+webhooks)
### `24-more-github-features/` — Copilot, Gists, Models, Packages, Marketplace, Codespaces, Education, Security (Dependabot/scan), Sponsors, Student Pack, Classroom
### `25-github-pages/` — deploy static, CNAME custom domain, Jekyll/Actions SSG
