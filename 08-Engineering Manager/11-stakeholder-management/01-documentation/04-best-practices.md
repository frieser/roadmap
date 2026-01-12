## Summary
Documentation Best Practices ensure that the effort spent writing docs translates into value for the reader. Good docs are concise, searchable, up-to-date, and audience-aware.

## Detailed Explanation
Writing is a user experience (UX) design task. The user is the reader.

### Golden Rules
1.  **Audience First**: Are you writing for a junior dev or the CTO?
2.  **Code Blocks**: Must be copy-pasteable and actually work.
3.  **Visuals**: A diagram is worth 1000 words.
4.  **Ownership**: Every page needs an "Owner" responsible for updates.

## Go Code Example
Modeling a `DocLinter` that checks for common best practice violations (e.g., missing code block languages, broken links).

```go
package main

import (
	"fmt"
	"strings"
)

type DocPage struct {
	Content string
	Links   []string
}

func (d DocPage) Lint() []string {
	var errors []string

	// Check 1: Code blocks must specify language
	if strings.Contains(d.Content, "```\n") {
		errors = append(errors, "Found code block without language tag")
	}

	// Check 2: No 'Click Here' links (accessibility)
	for _, link := range d.Links {
		if strings.ToLower(link) == "click here" {
			errors = append(errors, "Avoid 'Click Here' links; use descriptive text")
		}
	}

	return errors
}

func main() {
	badDoc := DocPage{
		Content: "# Setup\n```\ngo run main.go\n```",
		Links:   []string{"click here"},
	}

	issues := badDoc.Lint()
	fmt.Printf("Linting Issues Found: %d\n%v\n", len(issues), issues)
}
```

## Interview Questions
**Q: What is the most common mistake in technical documentation?**
**A:** Assuming knowledge. Writers often skip the "prerequisites" because they know them by heart, leaving new readers completely stuck on Step 1.

**Q: How do you measure documentation quality?**
**A:** Proxy metrics: Reduced support tickets, faster onboarding time for new hires, and direct feedback ("Was this helpful?" buttons).
