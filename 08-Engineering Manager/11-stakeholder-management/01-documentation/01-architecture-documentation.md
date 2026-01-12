## Summary
Architecture Documentation serves as the map for the system territory. It explains the "Big Picture" structure, component interactions, and high-level design choices. Without it, every new engineer is flying blind, and every change carries hidden risks of breaking dependencies.

## Detailed Explanation
Good architecture docs are "Living Documents" but shouldn't be too granular (code changes faster than docs).

### Key Models
1.  **C4 Model**: Context, Containers, Components, Code. Start high, zoom in.
2.  **System Context**: Who uses the system? (Users, other systems).
3.  **Data Flow**: Where does PII go? Where is the source of truth?

## Go Code Example
Modeling a `DiagramGenerator` that parses Go structs to auto-generate PlantUML or Mermaid context diagrams (Documentation as Code).

```go
package main

import (
	"fmt"
	"reflect"
)

type Service struct {
	Name         string
	Dependencies []string
}

type OrderService struct {
	PaymentService string
	InventorySvc   string
}

func GenerateMermaid(service interface{}) {
	t := reflect.TypeOf(service)
	serviceName := t.Name()

	fmt.Println("graph TD")
	for i := 0; i < t.NumField(); i++ {
		field := t.Field(i)
		fmt.Printf("    %s --> %s\n", serviceName, field.Name)
	}
}

func main() {
	// Documentation is generated FROM code, ensuring accuracy
	svc := OrderService{}
	GenerateMermaid(svc)
}
```

## Interview Questions
**Q: How do you keep architecture documentation from getting stale?**
**A:** I treat docs as code. Architecture diagrams are generated from source where possible, or stored as MermaidJS in the repo. Reviewing the "docs diff" is a mandatory part of the Pull Request for any architectural change.

**Q: What is the C4 model?**
**A:** It's a way to document software architecture at different zoom levels: Context (System level), Container (Applications/Databases), Component (Internal structure), and Code (Classes/Interfaces).
