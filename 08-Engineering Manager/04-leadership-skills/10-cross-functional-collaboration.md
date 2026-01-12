## Summary
Cross-functional collaboration is the ability of an engineering team to work effectively with other departments (Product, Design, Marketing, Sales, Support). Engineering does not exist in a vacuum; it exists to solve business problems. Breaking down silos and establishing a shared vocabulary are key to shipping successful products.

## Detailed Explanation
### The Triad (Product, Design, Engineering)
The core product team.
*   **Product**: Defines the "Why" and "What" (Business value).
*   **Design**: Defines the "How it looks/feels" (User Experience).
*   **Engineering**: Defines the "How it works" (Feasibility/Implementation).
Collaboration works best when these three are involved *early* (Discovery phase), not just at handoff.

### Bridging the Gap
*   **Empathy**: Understand their KPIs. (Sales needs features to sell; Support needs fewer bugs).
*   **Shared Goals**: OKRs should be shared across functions, not siloed.
*   **Ubiquitous Language**: Use the same terms for domain concepts.

### Go Code Example: The Collaboration Interface
This code models a project lifecycle that enforces sign-offs from different departments, ensuring collaboration isn't skipped.

```go
package leadership

import "fmt"

// Department represents a functional area
type Department interface {
	Review(spec string) bool
	Name() string
}

type Product struct{}
func (p Product) Review(spec string) bool { return true } // Product usually approves
func (p Product) Name() string { return "Product" }

type Design struct{}
func (d Design) Review(spec string) bool { return true }
func (d Design) Name() string { return "Design" }

type Security struct{}
func (s Security) Review(spec string) bool { 
	if spec == "Unsafe" { return false }
	return true 
}
func (s Security) Name() string { return "Security" }

// Project requires cross-functional sign-off
type Project struct {
	Spec string
	Approvals map[string]bool
}

func (p *Project) GetSignOffs(deps []Department) {
	p.Approvals = make(map[string]bool)
	allApproved := true
	
	for _, d := range deps {
		approved := d.Review(p.Spec)
		p.Approvals[d.Name()] = approved
		if !approved {
			fmt.Printf("❌ %s rejected the spec.\n", d.Name())
			allApproved = false
		} else {
			fmt.Printf("✅ %s approved.\n", d.Name())
		}
	}

	if allApproved {
		fmt.Println("🚀 Project ready for development!")
	} else {
		fmt.Println("⚠️  Project blocked. Collaboration required.")
	}
}

func main() {
	proj := Project{Spec: "New Login Flow"}
	collaborators := []Department{Product{}, Design{}, Security{}}
	
	proj.GetSignOffs(collaborators)
}
```

## Interview Questions
**Q: How do you handle a conflict with a Product Manager?**
**A:** I assume positive intent. We both want the product to succeed. I clarify if the conflict is about value (Product domain) or feasibility (Engineering domain). If they push for a feature that is technically risky, I explain the risk in business terms ("This will slow down the site by 20%"). If we still disagree, we might agree to a "disagree and commit" approach or escalate if the risk is existential.

**Q: How do you work with QA/Support?**
**A:** I view Support as the "voice of the customer." I set up regular syncs with Support leads to review top ticket drivers. I invite QA to design reviews so they can write test plans *before* code is written. I ensure engineers respect QA findings and don't treat them as "blockers" but as "quality partners."

**Q: Sales sold a feature that doesn't exist. What do you do?**
**A:** I stay calm. I talk to the Sales rep to understand the promise and the customer need. Then I assess if we can build a "MVP" version that satisfies the contract without derailing the roadmap. I also work with Sales Leadership to improve the process so they check with Engineering *before* promising dates/features in the future.
