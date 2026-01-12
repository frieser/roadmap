# Common API Vulnerabilities

## Summary
API security vulnerabilities often stem from flaws in how an API handles authentication, authorization, and data processing. The **OWASP API Security Top 10** lists the most critical risks, including Broken Object Level Authorization (BOLA), Broken User Authentication, and Excessive Data Exposure. Understanding these vulnerabilities is crucial for developers to build robust systems that protect sensitive user data and ensure service integrity.

## Detailed Explanation

### 1. Broken Object Level Authorization (BOLA/IDOR)
**The Problem**: An API exposes an endpoint that takes an ID (e.g., `/users/123/orders`), but fails to check if the *authenticated user* actually has permission to access *that specific object*. A user with ID 5 can access the data of ID 123 simply by changing the number in the URL.

#### Vulnerable Code (Go)
```go
func GetOrder(w http.ResponseWriter, r *http.Request) {
    // ❌ VULNERABLE: No check if userID from token matches orderOwnerID
    vars := mux.Vars(r)
    orderID := vars["id"]
    
    var order Order
    // Select * from orders where id = orderID
    db.First(&order, orderID) 
    
    json.NewEncoder(w).Encode(order)
}
```

#### Secure Code (Go)
```go
func GetOrder(w http.ResponseWriter, r *http.Request) {
    // ✅ SECURE: Verify ownership
    userID := r.Context().Value("user_id").(int)
    vars := mux.Vars(r)
    orderID := vars["id"]
    
    var order Order
    // Explicitly check ownership in query
    if err := db.Where("id = ? AND user_id = ?", orderID, userID).First(&order).Error; err != nil {
        http.Error(w, "Not Found", http.StatusNotFound)
        return
    }
    
    json.NewEncoder(w).Encode(order)
}
```

### 2. Broken User Authentication
**The Problem**: Weak authentication mechanisms allow attackers to compromise tokens or guess credentials. Examples include allowing weak passwords, lack of rate limiting on login, or not validating token signatures correctly.

#### Mitigation Strategy
*   Use strong, standard auth libraries (like `golang-jwt`).
*   Implement strict rate limiting on `/login` endpoints.
*   Enforce password complexity and rotation policies.

### 3. Excessive Data Exposure
**The Problem**: The API returns more data than the client needs (e.g., returning a full User object including `password_hash` and `ssn`), relying on the client-side code to filter it out. Attackers can inspect the raw JSON response to steal sensitive info.

#### Vulnerable Code (Go)
```go
type User struct {
    ID       int    `json:"id"`
    Username string `json:"username"`
    Password string `json:"password"` // ❌ Exposed in JSON!
    Email    string `json:"email"`
}

func GetUser(w http.ResponseWriter, r *http.Request) {
    var user User
    db.First(&user, 1)
    json.NewEncoder(w).Encode(user) // Sends everything, including password
}
```

#### Secure Code (Go)
```go
type UserDTO struct {
    ID       int    `json:"id"`
    Username string `json:"username"`
    // Password field is omitted entirely
}

func GetUser(w http.ResponseWriter, r *http.Request) {
    var user User
    db.First(&user, 1)
    
    // Map to DTO
    response := UserDTO{
        ID:       user.ID,
        Username: user.Username,
    }
    
    json.NewEncoder(w).Encode(response)
}
```

### 4. Lack of Resources & Rate Limiting
**The Problem**: The API does not restrict the number or frequency of requests from a client. Attackers can launch DoS attacks or brute-force credentials.

#### Mitigation in Go
Use a middleware like `golang.org/x/time/rate`.

```go
var limiter = rate.NewLimiter(1, 3) // 1 req/sec, burst of 3

func limitMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if !limiter.Allow() {
            http.Error(w, "Too Many Requests", http.StatusTooManyRequests)
            return
        }
        next.ServeHTTP(w, r)
    })
}
```

### 5. Injection (SQL, NoSQL, Command)
**The Problem**: Untrusted data is sent to an interpreter as part of a command or query. The interpreter executes unintended commands.

#### Vulnerable Code (Go)
```go
// ❌ VULNERABLE: String concatenation
query := fmt.Sprintf("SELECT * FROM users WHERE name = '%s'", r.FormValue("name"))
db.Exec(query)
```

#### Secure Code (Go)
```go
// ✅ SECURE: Parameterized query
name := r.FormValue("name")
db.Exec("SELECT * FROM users WHERE name = ?", name)
```

### Visualizing BOLA

```mermaid
sequenceDiagram
    participant Attacker (ID: 5)
    participant API
    participant DB

    Attacker->>API: GET /orders/123 (Auth Token: User 5)
    Note right of API: BOLA Vulnerability! <br/>API only checks if Token is valid,<br/>NOT if User 5 owns Order 123.
    API->>DB: SELECT * FROM orders WHERE id = 123
    DB-->>API: Returns Order 123 Data
    API-->>Attacker: 200 OK { "id": 123, "cc": "4111...", "owner": 99 }
    Note right of Attacker: Attacker steals data of User 99
```

## Interview Questions

**Q: What is BOLA (Broken Object Level Authorization) and how do you prevent it?**
**A:** BOLA occurs when an authenticated user accesses resources they don't own by manipulating the resource ID (e.g., changing `/users/1` to `/users/2`). To prevent it, always validate that the `current_user.ID` from the auth token matches the `resource.OwnerID` in the database before returning data.

**Q: Why is Excessive Data Exposure dangerous if the frontend filters the data?**
**A:** Because an attacker can bypass the frontend and call the API directly (e.g., using Postman or curl). If the API sends sensitive fields like `password_hash` or `is_admin`, the attacker can see them in the raw HTTP response, leading to credential theft or privilege escalation.

**Q: How does Mass Assignment vulnerability work?**
**A:** Mass Assignment happens when an API endpoint binds client input directly to internal objects without filtering. For example, if a user sends `{"is_admin": true}` during a profile update and the backend automatically maps this to the User model, a standard user could promote themselves to admin. Fix it by using strict DTOs or whitelisting updatable fields.

**Q: Explain the difference between Authentication and Authorization failures.**
**A:** **Authentication** failures mean the system cannot correctly identify who the user is (e.g., weak passwords, credential stuffing). **Authorization** failures mean the system identified the user correctly but failed to restrict their access permissions (e.g., BOLA, allowing a regular user to access admin endpoints).
