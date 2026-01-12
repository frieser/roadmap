## Summary
Scope Management is the process of defining what is *included* and what is *excluded* from a project. It ensures that the project team focuses only on approved work, preventing "Scope Creep"—the uncontrolled expansion of requirements without adjustments to time, cost, or resources.

## Detailed Explanation
Effective scope management requires clear documentation and a formal change control process. It protects the team from burnout and the project from budget overruns.

### Components
1.  **Scope Planning**: Creating a Scope Management Plan.
2.  **Scope Definition**: Detailed description of product and project deliverables.
3.  **WBS (Work Breakdown Structure)**: Decomposing deliverables into work packages.
4.  **Scope Verification**: Formal acceptance of deliverables by stakeholders.
5.  **Scope Control**: Monitoring status and managing changes.

### Scope Creep
Common causes include:
*   Ambiguous requirements.
*   "Gold-plating" (engineers adding cool but unrequested features).
*   Direct stakeholder requests bypassing the Product Owner.

## Go Code Example
A system to manage a project's scope, including a mechanism to reject "Gold Plating" and formally handle Change Requests.

```go
package main

import (
	"fmt"
)

type Feature struct {
	Name     string
	Points   int
	Approved bool
}

type ProjectScope struct {
	MaxPoints      int
	CurrentPoints  int
	Features       []Feature
	ChangeLog      []string
}

// AddFeature attempts to add a feature to the scope
func (ps *ProjectScope) AddFeature(f Feature, isChangeRequest bool) error {
	// Protection against Gold Plating
	if !f.Approved {
		return fmt.Errorf("REJECTED: Feature '%s' is not approved (Gold Plating detected)", f.Name)
	}

	// Protection against Scope Creep (Capacity check)
	if ps.CurrentPoints+f.Points > ps.MaxPoints {
		if isChangeRequest {
			// Formal change request logic would go here (e.g., increase budget/time)
			return fmt.Errorf("HALT: Feature '%s' exceeds capacity. Change Request required to increase MaxPoints", f.Name)
		}
		return fmt.Errorf("REJECTED: Feature '%s' exceeds project capacity", f.Name)
	}

	ps.Features = append(ps.Features, f)
	ps.CurrentPoints += f.Points
	ps.ChangeLog = append(ps.ChangeLog, fmt.Sprintf("Added: %s (%d pts)", f.Name, f.Points))
	return nil
}

func main() {
	scope := ProjectScope{MaxPoints: 20, CurrentPoints: 0}

	// 1. Valid Feature
	err := scope.AddFeature(Feature{Name: "Login", Points: 5, Approved: true}, false)
	if err != nil { fmt.Println(err) }

	// 2. Gold Plating attempt (Unapproved)
	err = scope.AddFeature(Feature{Name: "Dark Mode Animation", Points: 3, Approved: false}, false)
	if err != nil { fmt.Println(err) }

	// 3. Scope Creep (Approved but exceeds capacity)
	// Assume we filled the scope
	scope.CurrentPoints = 18 
	err = scope.AddFeature(Feature{Name: "Reporting Dashboard", Points: 10, Approved: true}, true)
	if err != nil { fmt.Println(err) }
	
	fmt.Printf("Current Scope: %d/%d points\n", scope.CurrentPoints, scope.MaxPoints)
}
```

## Interview Questions
**Q: What is Scope Creep and how do you prevent it?**
**A:** Scope Creep is the unauthorized addition of work. I prevent it by having a signed-off requirements document or backlog at the start, ensuring a rigorous Change Control Process where every new request is evaluated for impact on cost/timeline, and empowering the team to say "no" or "not yet" to off-the-books requests.

**Q: How do you handle a stakeholder who insists on adding a feature late in the project?**
**A:** I don't say "no," I say "yes, but." "Yes, we can add this, but it will delay the release by X days or we must remove feature Y to accommodate it." I present the trade-offs so the stakeholder makes an informed business decision.

**Q: What is the difference between Product Scope and Project Scope?**
**A:** Product Scope refers to the features and functions that characterize the product (what it *does*). Project Scope includes the work required to deliver the product (how we *build* it, including testing, meetings, and deployment).
