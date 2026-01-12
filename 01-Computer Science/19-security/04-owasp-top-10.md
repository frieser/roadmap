---
---

## Summary
The **OWASP Top 10** is a standard awareness document for developers and web application security professionals. Managed by the **Open Web Application Security Project (OWASP)**, it represents a broad consensus on the most critical security risks to web applications. The list is typically updated every 3-4 years (e.g., 2017, 2021, and the recent 2025 release candidate) based on extensive data analysis from security firms and community surveys. Its goal is to provide a baseline for secure coding and a starting point for organizations to prioritize their security efforts.

## Detailed Explanation

### Comparison: 2021 vs. 2025 (Release Candidate)
The 2025 edition reflects a shift toward **Supply Chain security**, **automated misconfiguration detection**, and **resilience against exceptional conditions**.

| Rank | OWASP Top 10: 2021 | OWASP Top 10: 2025 (RC) |
| :--- | :--- | :--- |
| **A01** | **Broken Access Control** | **Broken Access Control** |
| **A02** | Cryptographic Failures | **Security Misconfiguration** (Up from #5) |
| **A03** | Injection | **Software Supply Chain Failures** (New/Consolidated) |
| **A04** | Insecure Design | Cryptographic Failures (Down from #2) |
| **A05** | Security Misconfiguration | Injection (Down from #3) |
| **A06** | Vulnerable/Outdated Components | Insecure Design (Down from #4) |
| **A07** | Identification/Auth Failures | Authentication Failures |
| **A08** | Software/Data Integrity Failures | Software or Data Integrity Failures |
| **A09** | Security Logging/Monitoring | Security Logging and Alerting Failures |
| **A10** | Server-Side Request Forgery (SSRF) | **Mishandling of Exceptional Conditions** (New) |

---

### Deep Dive: SQL Injection (SQLi)
*   **Mechanism**: SQL Injection occurs when untrusted data is sent to an interpreter as part of a command or query. An attacker can supply malicious SQL snippets that alter the intended query logic, potentially leading to unauthorized data access, modification, or even administrative control over the database.
*   **Example**: 
    `query := "SELECT * FROM users WHERE username = '" + userInput + "';"`
    If `userInput` is `' OR '1'='1`, the query becomes `SELECT * FROM users WHERE username = '' OR '1'='1';`, returning all users.
*   **Fix**: Use **Parameterized Queries (Prepared Statements)**. This ensures the database treats the input as data rather than executable code.

### Deep Dive: Cross-Site Scripting (XSS)
*   **Mechanism**: XSS allows attackers to execute malicious scripts in the victim's browser. It happens when an application includes untrusted data in a web page without proper validation or escaping.
*   **Types**:
    1.  **Stored XSS**: The script is permanently stored on the server (e.g., in a database or comment field).
    2.  **Reflected XSS**: The script is "reflected" off a web server (e.g., in a URL parameter or error message).
    3.  **DOM-based XSS**: The vulnerability exists in client-side code rather than server-side code.
*   **Fix**: **Context-aware output encoding**. Encode data based on where it will be placed (HTML body, attribute, JavaScript variable, or CSS).

### Deep Dive: Software Supply Chain Failures (A03:2025)
*   **Mechanism**: Risks stemming from third-party libraries, tools, and CI/CD pipelines. This includes using components with known vulnerabilities (formerly A06:2021) and compromises in the build/distribution process (like the SolarWinds hack).
*   **Fix**: Maintain a **Software Bill of Materials (SBOM)**, use dependency scanning tools (e.g., `govulncheck`), and verify package signatures.

---

## Go Context

Go provides powerful built-in tools to mitigate several OWASP Top 10 risks by default.

### 1. SQL Injection Prevention
The `database/sql` package promotes the use of placeholders (`?` for MySQL/SQL Server, `$1`, `$2` for PostgreSQL). These are automatically handled as prepared statements.

```go
// GOOD: Parameterized query
id := "123"
var username string
err := db.QueryRow("SELECT username FROM users WHERE id = $1", id).Scan(&username)
```

### 2. XSS Prevention
The `html/template` package in Go's standard library implements **context-aware auto-escaping**. It understands whether data is being placed in an HTML tag, an attribute, or a script block and encodes it accordingly.

```go
import "html/template"

func handler(w http.ResponseWriter, r *http.Request) {
    t := template.Must(template.New("web").Parse("<p>Hello, {{.}}</p>"))
    // If input is "<script>alert(1)</script>", it is rendered as "&lt;script&gt;..."
    t.Execute(w, r.URL.Query().Get("name"))
}
```

### 3. Supply Chain Security
Go's module system (`go.mod`) uses `go.sum` to ensure that dependencies are not tampered with. Additionally, `govulncheck` is the official tool to find known vulnerabilities in your dependency graph.

```bash
# Check for vulnerabilities in your Go project
govulncheck ./...
```

---

## Interview Questions

**Q1: What is the most significant change in the OWASP Top 10 2025 compared to 2021?**
**A:** The elevation of **Security Misconfiguration** to #2 and the introduction of **Software Supply Chain Failures** as a top-tier category (A03). This reflects the industry's shift from focusing solely on code vulnerabilities to securing the entire ecosystem, including CI/CD and dependencies.

**Q2: How does Go's `html/template` differ from `text/template` regarding security?**
**A:** `html/template` provides context-aware auto-escaping, which prevents XSS by encoding data based on its location in the HTML. `text/template` does not perform any escaping and is only suitable for non-HTML output where XSS isn't a risk.

**Q3: Explain "Broken Access Control" with a real-world example.**
**A:** It occurs when an application fails to enforce authorization. For example, if an authenticated user can access another user's private data by simply changing an ID in the URL (e.g., `/api/orders/101` to `/api/orders/102`), it's a "Horizontal Privilege Escalation" flaw.

**Q4: What is SSRF and how do you mitigate it?**
**A:** Server-Side Request Forgery happens when an attacker forces a server to make requests to unintended locations (often internal services like `localhost` or cloud metadata endpoints). Mitigation includes using allow-lists for URLs, validating hostnames/IPs, and disabling HTTP redirections.

**Q5: What is a Software Bill of Materials (SBOM)?**
**A:** An SBOM is a formal, machine-readable record of all the components, libraries, and dependencies used in building a software product. It is critical for tracking and responding to vulnerabilities in the supply chain (A03:2025).

---

## Diagram: Injection Attack Mechanism

```mermaid
graph TD
    User[Attacker/User] -->|1. Input: ' OR 1=1| App[Web Application]
    App -->|2. Insecure Query: SELECT... WHERE id='' OR 1=1| DB[(Database)]
    DB -->|3. Returns ALL records| App
    App -->|4. Sensitive Data Leak| User

    subgraph "Mitigation: Prepared Statement"
    App2[Web Application] -- "3. Query: SELECT... WHERE id=?" --> DB2[(Database)]
    App2 -- "4. Param: ' OR 1=1" --> DB2
    DB2 -->|5. Treats input as Literal String| DB2
    DB2 -->|6. Returns 0 records (safe)| App2
    end
```
