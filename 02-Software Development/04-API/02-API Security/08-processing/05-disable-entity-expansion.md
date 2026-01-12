#API #Security #DoS
---
---

# Disable Entity Expansion (Billion Laughs Attack)

## Summary
The **Billion Laughs Attack** (Exponential Entity Expansion) is a Denial-of-Service (DoS) attack targeting XML parsers. By nesting internal XML entities recursively, an attacker can cause a tiny payload (< 1KB) to expand into Gigabytes of memory usage, crashing the server. Prevention requires strict limits on entity expansion depth and total document size, or disabling DTDs entirely.

## Detailed Explanation

### 1. The Vulnerability
The attack exploits the macro-like behavior of XML entities.
*   **The Mechanism**: An attacker defines an entity `&lol;` as "lol". Then `&lol1;` as ten `&lol;`s. Then `&lol2;` as ten `&lol1;`s.
*   **Exponential Growth**: By level 9 (`&lol9;`), the parser attempts to create $10^9$ (1,000,000,000) copies of the string "lol".
*   **Impact**: CPU spikes to 100% and RAM is exhausted immediately (OOM Kill), taking down the service.

### 2. Quadratic Blowup
A variation where instead of nesting, a single large entity (e.g., 50MB string) is referenced thousands of times. It consumes memory linearly but effectively enough to crash systems.

### 3. Prevention Strategies
*   **Disable DTDs**: The root cause is the ability to define custom entities in the DTD. Disabling `DOCTYPE` support prevents the attack.
*   **Limit Expansion**: If DTDs are needed, configure the parser to limit recursion depth (e.g., max 10 levels) and expanded size.
*   **Input Size Limits**: Reject any XML body larger than a reasonable limit (e.g., 1MB) using `io.LimitReader`.

---

## Go (Golang) Application

### Safe Implementation
Go's `encoding/xml` is resilient but resource exhaustion is still possible via massive payloads. We use `io.LimitReader` and ensure custom entities are restricted.

```go
package main

import (
	"encoding/xml"
	"fmt"
	"io"
	"net/http"
	"strings"
)

func XMLHandler(w http.ResponseWriter, r *http.Request) {
	// 1. DEFENSE: Limit the total request body size (e.g., 1MB)
	// This prevents the server from reading a massive stream
	safeReader := io.LimitReader(r.Body, 1024*1024)

	decoder := xml.NewDecoder(safeReader)

	// 2. DEFENSE: encoding/xml does not expand custom entities from DTDs by default.
	// It only supports standard entities: lt, gt, amp, apos, quot.
	// Do NOT populate decoder.Entity with untrusted maps.

	for {
		t, err := decoder.Token()
		if err == io.EOF {
			break
		}
		if err != nil {
			http.Error(w, "Invalid XML", http.StatusBadRequest)
			return
		}

		// Inspect tokens safely...
		switch tok := t.(type) {
		case xml.StartElement:
			fmt.Printf("Start: %s\n", tok.Name.Local)
		}
	}
}
```

---

## Interview Questions

**Q1: What is the difference between XXE and the Billion Laughs attack?**
**A:** **XXE** (XML External Entity) targets **Confidentiality** (stealing files) and uses *External* entities (`SYSTEM "file://"`). **Billion Laughs** targets **Availability** (DoS) and uses *Internal* recursive entities to exhaust memory.

**Q2: How small can a Billion Laughs payload be?**
**A:** Extremely small. A payload of less than 1 kilobyte can expand to occupy 3 gigabytes of RAM. This high compression ratio makes it a dangerous asymmetric attack (low cost for attacker, high cost for defender).

**Q3: How does disabling DTDs prevent this attack?**
**A:** The attack requires the definition of custom entities (`<!ENTITY ...>`) which resides inside the Document Type Definition (DTD) block. If the parser is configured to ignore or reject DTDs (`<!DOCTYPE>`), the entities are never defined, and the attack fails.

**Q4: Is it enough to just limit the file upload size?**
**A:** It helps, but not completely. Since the payload is so small (< 1KB), standard file size limits (e.g., 10MB) won't catch it. You specifically need to limit the **expanded** memory size or the recursion depth of the parser.
