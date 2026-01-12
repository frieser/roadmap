#API #Security #XXE
---
---

# Disable XML Entity Parsing (XXE)

## Summary
**XML External Entity (XXE)** injection is a critical vulnerability where an attacker exploits an insecurely configured XML parser. By defining a malicious external entity, an attacker can trick the parser into reading sensitive local files (like `/etc/passwd`), scanning internal network ports (SSRF), or causing a Denial of Service. The fix is to strictly disable DTDs (Document Type Definitions) and External Entity resolution.

## Detailed Explanation

### 1. The Vulnerability
XML allows the definition of **Entities**, which act like variables. An **External Entity** is one that fetches its value from a URI.
*   **Attack Payload**:
    ```xml
    <!DOCTYPE foo [
      <!ENTITY xxe SYSTEM "file:///etc/passwd">
    ]>
    <data>&xxe;</data>
    ```
*   **Execution**: If the parser is configured to process DTDs (`<!DOCTYPE...>`), it sees the `SYSTEM` keyword, opens the local file `/etc/passwd`, reads its content, and replaces `&xxe;` with that content. The API then returns the password file in the response.

### 2. Impact
*   **Confidentiality**: Reading server files (SSH keys, config files, source code).
*   **SSRF**: Requesting internal URLs (`http://169.254.169.254/latest/meta-data/`) to steal cloud credentials.
*   **Availability**: "Billion Laughs" attack (DoS).

### 3. Prevention Strategies
*   **Disable DTDs**: Completely forbid the `<!DOCTYPE>` declaration.
*   **Disable External Entities**: If DTDs are needed for structure, ensure `ExternalEnts` are turned off.
*   **Use JSON**: Prefer JSON APIs, which are immune to XXE.

---

## Go (Golang) Application

### Safe XML Parsing
The standard `encoding/xml` library in Go is **safe by default**. It ignores DTDs and does not process external entities unless you manually build a mechanism to do so.

```go
package main

import (
	"encoding/xml"
	"fmt"
	"strings"
)

type Data struct {
	Content string `xml:",innerxml"`
}

func main() {
	// Malicious Payload trying to read /etc/passwd
	payload := `
	<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
	<data>&xxe;</data>`

	var d Data
	// xml.Unmarshal ignores the DOCTYPE and the entity reference
	err := xml.Unmarshal([]byte(payload), &d)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}

	// Output will contain the literal "&xxe;" or be empty, 
	// NOT the contents of /etc/passwd
	fmt.Printf("Parsed Content: %s\n", d.Content)
}
```

### Warning: 3rd Party Libs (libxml2)
If you use C-bindings (cgo) for XML parsing (e.g., wrappers around `libxml2` for performance), you **MUST** explicitly disable network access and entity substitution.
```go
// Conceptual Example for libxml2 wrapper
parser.SetOptions(libxml2.XML_PARSE_NONET | libxml2.XML_PARSE_NOENT)
```

---

## Interview Questions

**Q1: How does an XXE attack allow an attacker to read files on the server?**
**A:** It abuses the XML standard's feature of "External Entities." The attacker defines an entity pointing to a `file://` URI. If the parser is insecure, it follows the URI, reads the file data, and substitutes the entity in the document with that data, which is then often returned to the user in the HTTP response.

**Q2: Is Go's `encoding/xml` vulnerable to XXE out of the box?**
**A:** No. Go's standard library parser does not support DTD entity resolution. It treats the DOCTYPE as a directive to be skipped or tokenized, but it does not execute the `SYSTEM` command to fetch external resources.

**Q3: What is "Blind XXE"?**
**A:** Blind XXE occurs when the application parses the XML but does *not* return the parsed content in the response. The attacker cannot see the file content directly. Instead, they must use "Out-of-Band" (OOB) techniques, forcing the server to send the data to an attacker-controlled server (e.g., `http://attacker.com/?data=CONTENT_OF_FILE`).

**Q4: Can XXE happen in JSON APIs?**
**A:** No. JSON format does not support DTDs or Entity references. However, if a JSON API accepts XML in a specific endpoint (e.g., legacy integration or file upload handling XML), that specific endpoint is vulnerable.
