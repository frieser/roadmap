## Summary
Knowledge Bases (KB) are centralized repositories of truth. They include wikis (Confluence), READMEs, and runbooks. The challenge is not creating them, but structuring and maintaining them so they don't become a "graveyard of information."

## Detailed Explanation
A KB must be searchable and curated.

### Structure
*   **Tree Structure**: Engineering > Backend > Service A.
*   **Tags**: Cross-cutting concerns like #onboarding, #incident.
*   **Search**: The primary UI. If search fails, the KB fails.

### Gardening
Assign "Gardeners" to prune old content. A smaller, accurate KB is better than a massive, contradictory one.

## Go Code Example
Modeling a `KBEntry` with content versioning and simplified diff tracking.

```go
package main

import "fmt"

type Version struct {
	Rev     int
	Content string
}

type KBEntry struct {
	Title    string
	Versions []Version
}

func (k *KBEntry) Update(newContent string) {
	nextRev := len(k.Versions) + 1
	k.Versions = append(k.Versions, Version{Rev: nextRev, Content: newContent})
	fmt.Printf("Updated '%s' to Rev %d\n", k.Title, nextRev)
}

func (k KBEntry) GetLatest() string {
	if len(k.Versions) == 0 {
		return ""
	}
	return k.Versions[len(k.Versions)-1].Content
}

func main() {
	page := KBEntry{Title: "How to SSH"}
	page.Update("Use key.pem")
	page.Update("Use key.pem with port 22")
	
	fmt.Println("Current Doc:", page.GetLatest())
}
```

## Interview Questions
**Q: Confluence vs. Markdown in Git. Which do you choose?**
**A:** Markdown in Git for anything technical (API specs, architecture, runbooks) because it lives with the code and uses PR workflow. Confluence for HR policies, broad roadmaps, and non-technical stakeholder docs.

**Q: How do you organize a chaotic wiki?**
**A:** I start by archiving. Move everything older than 1 year to an "Archive" folder. Then I create a top-level "Map" page with links to the 10 most important active docs.
