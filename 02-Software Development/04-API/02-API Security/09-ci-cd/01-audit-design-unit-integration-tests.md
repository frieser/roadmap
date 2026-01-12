#API #Security #Go
---
---

# Audit Design, Unit, and Integration Tests

Audit and testing in DevSecOps focus on **shifting security to the left** by identifying vulnerabilities early in the development lifecycle. This involves auditing the design (Threat Modeling) and implementing a tiered testing strategy (Security Testing Pyramid) from granular unit tests to broad integration tests.

## 1. Concept: Security Testing Pyramid
The Security Testing Pyramid for APIs adapts the classic testing pyramid to include security-specific verification:

*   **Unit Tests (Base)**: High-volume, fast tests focusing on individual functions.
    *   **Security Focus**: Input validation logic, cryptographic function correctness, data serialization/deserialization, and business logic authorization checks at the service level.
*   **Integration Tests (Middle)**: Testing interactions between components.
    *   **Security Focus**: Authentication middleware integration, Authorization (RBAC/ABAC) across layers, Database access controls, and Third-party API security.
*   **E2E / System Tests (Top)**: Testing the complete system from the outside.
    *   **Security Focus**: Dynamic Application Security Testing (DAST), penetration testing, and verifying security headers.

## 2. Strategy: Auditing the Design (Threat Modeling)
Before writing code, the API design must be audited for security flaws.

*   **Threat Modeling**: Using frameworks like **STRIDE** (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege) to identify potential attack vectors.
*   **Security Design Review**: Ensuring adherence to principles like **Least Privilege**, **Defense in Depth**, and **Zero Trust**.
*   **Auditing Entry Points**: Identifying all public endpoints, data formats, and authentication requirements.

## 3. Examples of Security-Specific Tests
*   **AuthZ Bypass**: Ensuring a user with `Role: User` cannot access `/admin` resources.
*   **IDOR (Insecure Direct Object Reference)**: Verifying that User A cannot access `/api/users/B/data` by simply changing the ID.
*   **Input Validation**: Testing against common injection attacks (SQLi, NoSQLi) and ensuring appropriate response codes (e.g., `400 Bad Request`).
*   **Rate Limiting**: Verifying that the API rejects excessive requests (e.g., `429 Too Many Requests`).
*   **Sensitive Data Leakage**: Asserting that error messages or responses do not contain PII or stack traces.

## 4. Go (Golang) Code Examples

### Security Unit Test (Asserting 403 Forbidden)
Testing that a handler correctly denies access when permissions are missing.

**Evidence** ([source](https://github.com/grafana/grafana/blob/main/pkg/api/admin_test.go)): Grafana uses table-driven tests to verify RBAC permissions for their API endpoints.

```go
func TestAPI_AdminGetSettings_AccessDenied(t *testing.T) {
    // Setup a mock recorder to capture the response
    w := httptest.NewRecorder()
    
    // Create a request to a protected endpoint
    req, _ := http.NewRequest("GET", "/api/admin/settings", nil)
    
    // Simulate a user without administrative permissions
    // In a real test, this would involve setting a context or mock session
    ctx := contextWithUser(req.Context(), &User{Role: "Viewer"})
    req = req.WithContext(ctx)

    // Execute the handler
    AdminGetSettingsHandler(w, req)

    // Assert that the response is 403 Forbidden
    if w.Code != http.StatusForbidden {
        t.Errorf("expected 403 Forbidden, got %d", w.Code)
    }
}
```

### Input Validation Test
Verifying that malicious input is rejected.

**Evidence** ([source](https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/apiserver/pkg/endpoints/handlers/responsewriters/errors_test.go)): Kubernetes tests ensure error handlers encode/escape malicious input like `<script>` in error messages.

```go
func TestInputValidation(t *testing.T) {
    cases := []struct {
        name     string
        input    string
        expected int
    }{
        {"Valid input", "normal_user", http.StatusOK},
        {"Malicious script", "<script>alert(1)</script>", http.StatusBadRequest},
        {"SQL Injection attempt", "' OR 1=1 --", http.StatusBadRequest},
    }

    for _, tc := range cases {
        t.Run(tc.name, func(t *testing.T) {
            err := ValidateUserInput(tc.input)
            if tc.expected == http.StatusBadRequest && err == nil {
                t.Errorf("expected error for input %q, but got nil", tc.input)
            }
        })
    }
}
```

## 5. Interview Preparation Questions

1.  **How do you test for IDOR (Insecure Direct Object Reference) in an API?**
    *   **Answer**: Perform a test where User A attempts to access a resource belonging to User B (e.g., `GET /api/account/USER_B_ID`). The test passes if the system returns a `403 Forbidden` or `404 Not Found`, even with User A's valid token.
2.  **What is the difference between DAST and SAST in API Security?**
    *   **Answer**: SAST analyzes source code for vulnerabilities (static) without running it. DAST tests the running API from the outside (dynamic), sending malicious payloads to discover runtime vulnerabilities like SQL injection.
3.  **Why is Threat Modeling considered "auditing the design"?**
    *   **Answer**: It identifies security risks at the architectural level *before* implementation, allowing developers to bake in controls (like specific validation or auth logic) when the cost of change is lowest.
4.  **How would you use Go's `httptest` to verify JWT protection?**
    *   **Answer**: Use `httptest.NewRecorder()` to make requests without a token (expect `401`), with an invalid token (expect `401`), and with a valid token (expect `200`).
