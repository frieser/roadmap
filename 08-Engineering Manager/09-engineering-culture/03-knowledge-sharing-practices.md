## Summary
Knowledge Sharing Practices are the mechanisms by which information moves from individual heads into the collective team consciousness. Effective sharing reduces silos, decreases the "bus factor," and accelerates onboarding. It encompasses documentation, code reviews, tech talks, and pair programming.

## Detailed Explanation
Knowledge hoarding is often unintentional, resulting from busy schedules or lack of process. A manager's job is to lower the barrier to sharing.

### Mechanisms
*   **Asynchronous**: Wikis (Confluence/Notion), Architecture Decision Records (ADRs), well-commented code, pull request descriptions.
*   **Synchronous**: Pair/Mob programming, "Brown Bag" sessions, incident reviews.

### Best Practices
1.  **Write it Down**: "If it's not written down, it didn't happen."
2.  **Make it Searchable**: Information is useless if it can't be found.
3.  **Keep it Fresh**: stale documentation is worse than no documentation. Implement "verify by" dates.
4.  **Reward Sharing**: Recognize good documentation and helpfulness in performance reviews.

## Go Code Example
Modeling a simple `KnowledgeBase` system that tracks article validity and searchability.

```go
package main

import (
	"fmt"
	"strings"
	"time"
)

type Article struct {
	ID        string
	Title     string
	Content   string
	Tags      []string
	LastUpdated time.Time
	ExpiresAt   time.Time
}

type KnowledgeBase struct {
	articles map[string]Article
}

func NewKnowledgeBase() *KnowledgeBase {
	return &KnowledgeBase{articles: make(map[string]Article)}
}

func (kb *KnowledgeBase) AddArticle(a Article) {
	// Default expiry to 6 months to force review
	if a.ExpiresAt.IsZero() {
		a.ExpiresAt = time.Now().AddDate(0, 6, 0)
	}
	kb.articles[a.ID] = a
}

func (kb *KnowledgeBase) Search(query string) []Article {
	var results []Article
	query = strings.ToLower(query)
	for _, a := range kb.articles {
		// Check validity
		if time.Now().After(a.ExpiresAt) {
			fmt.Printf("Warning: Skipping expired article: %s\n", a.Title)
			continue
		}
		// Simple keyword match
		if strings.Contains(strings.ToLower(a.Title), query) || 
		   strings.Contains(strings.ToLower(a.Content), query) {
			results = append(results, a)
		}
	}
	return results
}

func main() {
	kb := NewKnowledgeBase()
	kb.AddArticle(Article{
		ID: "1", Title: "Deployment Process", 
		Content: "Run kubectl apply...", 
		LastUpdated: time.Now(),
	})
	
	// Simulate search
	results := kb.Search("kubectl")
	fmt.Printf("Found %d valid articles.\n", len(results))
}
```

## Interview Questions
**Q: How do you handle a "hero" developer who solves everything but shares nothing?**
**A:** I coach them privately that their senior impact is defined by how much they scale the team, not just their individual output. I would require them to produce ADRs or mentor others as part of their OKRs.

**Q: What documentation strategy do you prefer?**
**A:** I prefer "documentation close to code" (like Markdown in the repo) for technical details because it evolves with the version control. For high-level processes, a wiki is fine, but it needs an owner.

**Q: How do you prevent documentation from going stale?**
**A:** I implement "expiry dates" on docs or a "documentation test" during onboarding—if a new hire finds a doc confusing or wrong, fixing it is their first task (and the team's responsibility to support).
