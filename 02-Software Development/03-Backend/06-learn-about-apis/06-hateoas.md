---
---

## Summary
HATEOAS (Hypermedia as the Engine of Application State) is a constraint of the REST application architecture. It requires the server to provide information to the client about how to interact with the API through hypermedia (links) included in the responses.

## Detailed Explanation
HATEOAS is often considered the "final level" (Level 3) of the Richardson Maturity Model.

### How it works
In a HATEOAS-compliant API, the client doesn't need to know the hardcoded URLs for every action. Instead, the server returns links along with the data. These links tell the client what actions are currently possible and where to go to perform them.

### Example JSON Response with Links
```json
{
    "id": 123,
    "name": "Jane Doe",
    "status": "active",
    "_links": {
        "self": { "href": "/users/123" },
        "deactivate": { "href": "/users/123/deactivate" },
        "update": { "href": "/users/123", "method": "PATCH" }
    }
}
```

### Benefits
- **Decoupling**: The server can change URL structures without breaking clients (as long as the client follows the links).
- **Discoverability**: The API becomes self-documenting for a client that knows how to parse the links.
- **Workflow Control**: The server only provides links for actions that are valid in the current state (e.g., no "cancel" link if the order is already shipped).

## Go Context
Implementing HATEOAS in Go usually involves adding a `Links` field to your response structs.

### Example: Struct with HATEOAS links
```go
package main

import (
	"encoding/json"
	"fmt"
)

type Link struct {
	Rel  string `json:"rel"`
	Href string `json:"href"`
}

type UserResponse struct {
	ID    int    `json:"id"`
	Email string `json:"email"`
	Links []Link `json:"links"`
}

func main() {
	res := UserResponse{
		ID:    1,
		Email: "user@example.com",
		Links: []Link{
			{Rel: "self", Href: "/users/1"},
			{Rel: "delete", Href: "/users/1"},
		},
	}

	data, _ := json.MarshalIndent(res, "", "  ")
	fmt.Println(string(data))
}
```

## Interview Questions
- **Q: What is the Richardson Maturity Model?**
- **A:** It is a way to grade APIs based on how "RESTful" they are. Level 0 is using HTTP as a transport, Level 1 uses Resources, Level 2 uses HTTP Verbs, and Level 3 is HATEOAS.

- **Q: Why is HATEOAS rarely fully implemented in the real world?**
- **A:** It adds complexity and payload size. Many developers find that well-documented resource paths (Level 2) are sufficient for most use cases, and standard client libraries often don't leverage hypermedia links automatically.

- **Q: How does HATEOAS help with API evolution?**
- **A:** Since clients find actions via links rather than hardcoded URLs, the server can change its internal routing or resource paths as long as it continues to provide the correct links under the same "rel" (relationship) tags.
