# AGENTS.md - Roadmap

## OVERVIEW
Technical learning roadmap: 2100+ atomic notes across 9 tracks, 9 levels deep. Mirrors roadmap.sh structure.

## STRUCTURE
```
Roadmap/
├── 01-Computer Science/      # Foundations: data structures, algorithms, complexity
├── 02-Software Development/  # Languages (Go, Python, Rust, Zig), APIs, DSA
├── 03-Software Architecture/ # Design patterns, clean code, architecture styles
├── 04-System Design/         # DevOps, cloud patterns, service mesh
├── 05-Tools/                 # Bash, Git, Docker, K8s, Terraform, Ansible, Linux
├── 06-Cloud/                 # AWS services deep dives
├── 07-AI/                    # ML, AI Engineer track
├── 09-Engineering Manager/   # Leadership, stakeholder mgmt, change management
└── Roadmap.sh.canvas         # Visual overview (Obsidian Canvas)
```

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Data structures | `01-Computer Science/02-data-structures/` |
| Algorithm patterns | `01-Computer Science/04-common-algorithms/` |
| Language roadmaps | `02-Software Development/01-Programming Languages/{Lang}/` |
| API design/security | `02-Software Development/05-API/` |
| Clean code principles | `03-Software Architecture/02-Software Design and Architecture/01-clean-code/` |
| Container orchestration | `05-Tools/Kubernetes/` |
| AWS services | `06-Cloud/AWS/` |
| Management skills | `09-Engineering Manager/` |

## CONVENTIONS (Roadmap-Specific)

### Numbering
- **Two-digit prefix**: `XX-name/` for directories, `XX-name.md` for files
- **Recursive**: Applied consistently through all 9 levels
- **Gaps allowed**: `08-` missing at top level (intentional)

### Naming
- **Kebab-case ONLY**: `01-adjacency-list.md`, `04-shell-fundamentals/`
- **All lowercase**: No Title Case in this hierarchy
- **English ONLY**: Technical content policy

### Language Wrappers (Inconsistent - Know Before Editing)
| Language | Uses `roadmap-project/` |
|----------|-------------------------|
| Python | Yes |
| Rust | Yes |
| Golang | No - content directly in folder |
| Zig | No - content directly in folder |

### File Structure
- YAML frontmatter: `---\n---` (placeholder)
- Single concept per file (atomic)
- No tags typically (relies on folder structure)

## ANTI-PATTERNS

| Forbidden | Why |
|-----------|-----|
| Title Case filenames | Breaks kebab-case consistency |
| Spaces in paths | CLI friction, linking issues |
| Skip XX- prefix | Breaks sort order |
| Duplicate topics across tracks | Already exists: CAP theorem in CS + System Design |
| Deep nesting without atomic split | Keep concepts single-file |

## KNOWN ISSUES
- `18-Version Changelog.md` in Golang breaks naming (should be kebab-case)
- Spaces in some parent dirs: `02-Software Development`, `04-System Design`
- `roadmap-project` wrapper inconsistently applied to languages

## NOTES
- Max depth: 9 levels (e.g., `05-Tools/Bash/04-shell-fundamentals/...`)
- ~280 directories at depth 6 (highest concentration)
- Canvas file provides visual navigation of entire structure
