# Technical Standards Setting

## Summary
Technical standards setting involves establishing, maintaining, and enforcing guidelines for code quality, architectural patterns, and engineering practices within an organization. It ensures consistency, maintainability, and scalability of the codebase while reducing cognitive load for developers. Effective standards are consensus-driven, clearly documented, and automated where possible.

## Detailed Explanation
Setting technical standards is a core responsibility of engineering leadership. It moves a team from ad-hoc decision-making to a predictable, high-quality engineering culture.

### Key Components
1.  **Coding Conventions**: Style guides (linting, formatting), naming conventions, and directory structures.
2.  **Architectural Patterns**: Standardizing on patterns like Clean Architecture, Hexagonal Architecture, or specific microservices patterns to ensure services look and behave similarly.
3.  **Technology Radar**: Defining which languages, frameworks, and datastores are "Adopt," "Trial," "Assess," or "Hold."
4.  **Review Process**: establishing how standards are proposed (e.g., RFCs - Request for Comments) and ratified.

### Implementation Strategy
- **Automation First**: If a standard can be enforced by a linter or CI check, it should be. Manual enforcement in code reviews is prone to error and friction.
- **Living Documents**: Standards should evolve. Create a lightweight RFC process for engineers to propose changes.
- **Golden Paths**: Create templates and libraries that make doing the "right thing" the easiest path (e.g., a service template that comes pre-configured with standard logging, metrics, and linting).

### Challenges
- **Over-standardization**: Stifling innovation by enforcing rigid rules where flexibility is needed.
- **Zombie Standards**: Rules that exist on a wiki but are ignored in practice.
- **Adoption**: Getting buy-in from the team is crucial; top-down mandates often fail without developer input.

## Go Code Example
The following example demonstrates a programmatic approach to enforcing standards. It represents a "Compliance Checker" that might run in a CI pipeline to ensure a Go project adheres to specific architectural rules, such as package import restrictions (e.g., "domain" layer cannot import "infrastructure").

```go
package main

import (
	"fmt"
	"strings"
)

// Rule defines a standard to be enforced
type Rule struct {
	Name        string
	Description string
	Check       func(packageName string, imports []string) error
}

// LinterConfig represents the configuration for our standards enforcer
type LinterConfig struct {
	Module string
	Rules  []Rule
}

// ArchitectureVerifier checks if code adheres to defined dependency rules
type ArchitectureVerifier struct {
	Config LinterConfig
}

// NewArchitectureVerifier creates a verifier with standard architectural rules
func NewArchitectureVerifier(moduleName string) *ArchitectureVerifier {
	return &ArchitectureVerifier{
		Config: LinterConfig{
			Module: moduleName,
			Rules: []Rule{
				{
					Name:        "Domain Isolation",
					Description: "Domain package should not import infrastructure",
					Check: func(pkg string, imports []string) error {
						if strings.Contains(pkg, "/domain") {
							for _, imp := range imports {
								if strings.Contains(imp, "/infrastructure") {
									return fmt.Errorf("violation: domain package imports infrastructure: %s", imp)
								}
							}
						}
						return nil
					},
				},
				{
					Name:        "No Relative Imports",
					Description: "Use absolute paths for imports",
					Check: func(pkg string, imports []string) error {
						for _, imp := range imports {
							if strings.HasPrefix(imp, ".") || strings.HasPrefix(imp, "..") {
								return fmt.Errorf("violation: relative import found: %s", imp)
							}
						}
						return nil
					},
				},
			},
		},
	}
}

// Run simulates checking files (in a real scenario, this would parse actual .go files)
func (av *ArchitectureVerifier) Run(simulatedFiles map[string][]string) []error {
	var errors []error
	fmt.Printf("Running Standards Check for module: %s\n", av.Config.Module)

	for pkg, imports := range simulatedFiles {
		for _, rule := range av.Config.Rules {
			if err := rule.Check(pkg, imports); err != nil {
				errors = append(errors, fmt.Errorf("[%s] in %s: %v", rule.Name, pkg, err))
			}
		}
	}
	return errors
}

func main() {
	verifier := NewArchitectureVerifier("github.com/org/payment-service")

	// Simulated codebase structure
	codebase := map[string][]string{
		"github.com/org/payment-service/domain/payment": {
			"time",
			"github.com/shopspring/decimal",
			// Violation: Domain importing infrastructure
			"github.com/org/payment-service/infrastructure/postgres", 
		},
		"github.com/org/payment-service/infrastructure/postgres": {
			"database/sql",
			"github.com/lib/pq",
		},
	}

	violations := verifier.Run(codebase)

	if len(violations) > 0 {
		fmt.Println("\n❌ Standards Violations Found:")
		for _, v := range violations {
			fmt.Println(v)
		}
	} else {
		fmt.Println("\n✅ All Standards Passed")
	}
}
```

## Interview Questions
1.  **How do you approach introducing a new technical standard to an existing team that might be resistant to change?**
    *   *Focus*: Change management, empathy, consensus building, RFC process.
2.  **Can you describe a time when a technical standard you implemented failed or caused friction? What did you learn?**
    *   *Focus*: Reflection, adaptability, understanding trade-offs between rigidity and velocity.
3.  **How do you balance "standardization" with "innovation"? When is it okay for a team to deviate from the paved path?**
    *   *Focus*: Governance models (e.g., "golden path"), handling exceptions, experimental phases.
4.  **What is your strategy for enforcing standards: automated tooling (linting/CI) vs. manual code review?**
    *   *Focus*: Automation preference, reducing human friction, using code review for architectural semantics rather than style.
