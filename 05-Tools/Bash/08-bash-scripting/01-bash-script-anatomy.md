---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Bash Script Anatomy

## Summary

A well-structured Bash script includes a shebang, strict mode, documentation, variables, functions, and main logic. Understanding script anatomy helps write maintainable, robust scripts that handle errors gracefully and follow best practices.

## Detailed Explanation

### Complete Script Template

```bash
#!/bin/bash
#
# Script Name: deploy.sh
# Description: Deploy application to production
# Author: Developer Name
# Date: 2024-01-15
# Version: 1.0.0
#
# Usage: deploy.sh [options] <environment>
#

#######################################
# Strict Mode - Exit on errors
#######################################
set -euo pipefail
IFS=$'\n\t'

#######################################
# Constants
#######################################
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"
readonly LOG_FILE="/var/log/${SCRIPT_NAME}.log"

#######################################
# Variables
#######################################
ENVIRONMENT=""
VERBOSE=false
DRY_RUN=false

#######################################
# Functions
#######################################

usage() {
    cat << EOF
Usage: ${SCRIPT_NAME} [OPTIONS] <environment>

Deploy application to specified environment.

Options:
    -h, --help      Show this help message
    -v, --verbose   Enable verbose output
    -n, --dry-run   Show what would be done
    
Arguments:
    environment     Target environment (dev|staging|prod)

Examples:
    ${SCRIPT_NAME} dev
    ${SCRIPT_NAME} -v prod
EOF
}

log() {
    local level="$1"
    shift
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [${level}] $*" | tee -a "$LOG_FILE"
}

info() { log "INFO" "$@"; }
warn() { log "WARN" "$@" >&2; }
error() { log "ERROR" "$@" >&2; }
die() { error "$@"; exit 1; }

cleanup() {
    # Cleanup code here
    info "Cleaning up..."
}

#######################################
# Argument Parsing
#######################################

parse_args() {
    while [[ $# -gt 0 ]]; do
        case "$1" in
            -h|--help)
                usage
                exit 0
                ;;
            -v|--verbose)
                VERBOSE=true
                shift
                ;;
            -n|--dry-run)
                DRY_RUN=true
                shift
                ;;
            -*)
                die "Unknown option: $1"
                ;;
            *)
                ENVIRONMENT="$1"
                shift
                ;;
        esac
    done
    
    # Validate required arguments
    [[ -z "${ENVIRONMENT}" ]] && die "Environment is required"
}

#######################################
# Main Logic
#######################################

main() {
    parse_args "$@"
    
    info "Starting deployment to ${ENVIRONMENT}"
    
    # Your logic here
    if [[ "$DRY_RUN" == true ]]; then
        info "DRY RUN: Would deploy to ${ENVIRONMENT}"
    else
        # Actual deployment
        info "Deploying..."
    fi
    
    info "Deployment complete"
}

#######################################
# Script Entry Point
#######################################

trap cleanup EXIT
main "$@"
```

### Key Components Explained

| Component | Purpose |
|-----------|---------|
| **Shebang** | `#!/bin/bash` - specifies interpreter |
| **Strict mode** | `set -euo pipefail` - catch errors |
| **Constants** | `readonly` variables that don't change |
| **Functions** | Reusable code blocks |
| **Argument parsing** | Handle command-line options |
| **Main function** | Entry point logic |
| **Trap** | Cleanup on exit |

### set -euo pipefail Explained

```bash
set -e          # Exit on any command failure
set -u          # Error on undefined variables
set -o pipefail # Pipeline fails if any command fails
IFS=$'\n\t'     # Safer word splitting
```

## Interview Questions

**Q: What is the purpose of `#!/bin/bash` at the top of a script?**
**A:** It's the shebang that tells the kernel which interpreter to use. Without it, the system uses the default shell which may not be Bash. Always include it for portability.

**Q: What does `set -euo pipefail` do?**
**A:** `-e` exits on error, `-u` errors on undefined variables, `-o pipefail` makes pipelines fail properly. Together they make scripts fail fast and catch bugs early.

**Q: How do you get the directory where a script is located?**
**A:** `SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"`. This handles symlinks and relative paths correctly.
