---
---

## Summary
The **Model-View-Controller (MVC)** pattern is a foundational architectural pattern that separates an application into three interconnected components: the **Model** (data and business logic), the **View** (presentation and user interface), and the **Controller** (handles input and updates the model/view). This separation concerns facilitates parallel development, maintainability, and testing by decoupling the user interface from the underlying logic.

## Detailed Explanation

### 1. The Three Components
*   **Model**: The brain of the application. It manages data, logic, and rules. It is independent of the user interface. In Go, this is typically represented by `structs` and methods that interact with the database.
*   **View**: The face of the application. It displays data to the user. In a web API context, the "View" might be the JSON response serialization. In a server-side rendered app, it's the HTML template.
*   **Controller**: The traffic cop. It accepts input (HTTP requests), commands the Model to change state or retrieve data, and decides which View to display. In Go, these are often your HTTP Handlers.

### 2. Interaction Flow
1.  **User** interacts with the View (e.g., clicks a button).
2.  **Controller** receives the input.
3.  **Controller** validates input and calls the **Model**.
4.  **Model** performs business logic/database operations and returns data.
5.  **Controller** selects a **View** to render the data.
6.  **View** generates the final output (HTML/JSON) for the user.

### 3. Benefits & Trade-offs
*   **Pros**: Separation of concerns, testability, simultaneous development (backend/frontend).
*   **Cons**: Can introduce complexity in small applications; strict separation can sometimes require excessive boilerplate.

## Go Code Example

In Go, MVC is often implemented using `structs` (Model), `html/template` or JSON marshaling (View), and Handler functions (Controller).

```go
package main

import (
	"encoding/json"
	"net/http"
)

// --- MODEL ---
// User represents the data structure and business logic capabilities
type User struct {
	ID    int    `json:"id"`
	Name  string `json:"name"`
	Email string `json:"email"`
}

// UserModel handles database interactions (mocked here)
type UserModel struct{}

func (m *UserModel) FindByID(id int) *User {
	// Simulate DB query
	if id == 1 {
		return &User{ID: 1, Name: "Alice", Email: "alice@example.com"}
	}
	return nil
}

// --- VIEW ---
// JSONView handles the presentation logic (serialization)
func JSONView(w http.ResponseWriter, data interface{}, status int) {
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(status)
	if err := json.NewEncoder(w).Encode(data); err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
	}
}

// --- CONTROLLER ---
// UserController handles the HTTP request flow
type UserController struct {
	Model *UserModel
}

func (c *UserController) GetUser(w http.ResponseWriter, r *http.Request) {
	// 1. Parse Input (Simulated)
	userID := 1 

	// 2. Interact with Model
	user := c.Model.FindByID(userID)

	if user == nil {
		JSONView(w, map[string]string{"error": "User not found"}, http.StatusNotFound)
		return
	}

	// 3. Render View
	JSONView(w, user, http.StatusOK)
}

func main() {
	// Initialize Model and Controller
	model := &UserModel{}
	controller := &UserController{Model: model}

	// Route
	http.HandleFunc("/user", controller.GetUser)
	
	// Start Server
	http.ListenAndServe(":8080", nil)
}
```

## Interview Questions

### Q: Does Go enforce MVC?
**A:** No, Go is opinionated about formatting but unopinionated about architecture. MVC is a pattern you *choose* to implement. Go's standard library `net/http` provides the primitives (Handlers) that naturally fit the Controller role, but it doesn't force a structure like Rails or Django.

### Q: What is the difference between "Fat Model, Skinny Controller" and the opposite?
**A:** "Fat Model, Skinny Controller" is generally preferred. It means putting most business logic in the Model (methods on structs/services) so it's reusable and testable. The Controller should only handle HTTP concerns (parsing request, validating input, calling service, rendering response). A "Fat Controller" leads to duplicated logic and hard-to-test handlers.

### Q: How does MVC apply to a Single Page Application (SPA) backend?
**A:** In an SPA context, the "View" is effectively removed from the backend. The backend becomes a REST/RPC API that returns data (JSON). The "View" logic (HTML rendering) moves entirely to the client-side (React/Vue). The backend retains the Model and Controller (API Handlers).
