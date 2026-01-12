---
---

## Summary
SOAP (Simple Object Access Protocol) is a messaging protocol specification for exchanging structured information in the implementation of web services. It uses XML for its message format and usually relies on other application layer protocols (HTTP, SMTP) for message negotiation and transmission.

## Detailed Explanation
SOAP was once the dominant standard for web services before the rise of REST. It is highly structured and strictly defined.

### Key Components
- **Envelope**: The core wrapper that identifies the XML document as a SOAP message.
- **Header**: Contains optional information like security credentials or routing data.
- **Body**: Contains the actual call and response information.
- **Fault**: Provides information about errors that occurred while processing the message.

### WSDL (Web Services Description Language)
SOAP services are usually described by a WSDL file. This is an XML-based file that defines exactly what operations the service supports, what parameters they take, and what they return. This allows for automated client generation.

### Why use SOAP today?
While "legacy" in many contexts, SOAP is still used in enterprise environments (banking, insurance) due to:
- **Built-in Security**: WS-Security standard.
- **ACID Compliance**: Supports formal transactions.
- **Protocol Independence**: Can run over HTTP, SMTP, TCP, etc.

## Go Context
Go doesn't have a "standard" SOAP library like it does for JSON. Developers usually use third-party libraries like `hooklift/gowsdl` or manually construct XML payloads.

### Example: Simple SOAP Request (Manual)
```go
package main

import (
	"bytes"
	"fmt"
	"net/http"
)

func main() {
	soapPayload := `
	<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:web="http://www.example.com/">
	   <soapenv:Header/>
	   <soapenv:Body>
	      <web:GetPrice>
	         <web:ItemID>12345</web:ItemID>
	      </web:GetPrice>
	   </soapenv:Body>
	</soapenv:Envelope>`

	client := &http.Client{}
	req, _ := http.NewRequest("POST", "http://example.com/soap-service", bytes.NewBufferString(soapPayload))
	req.Header.Set("Content-Type", "text/xml; charset=utf-8")
	req.Header.Set("SOAPAction", "GetPrice")

	resp, err := client.Do(req)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	defer resp.Body.Close()
	fmt.Println("Response Status:", resp.Status)
}
```

## Interview Questions
- **Q: What are the main differences between SOAP and REST?**
- **A:** SOAP is a protocol with strict standards, while REST is an architectural style. SOAP uses only XML, whereas REST can use JSON, XML, or others. SOAP has built-in error handling and security standards (WS-Security).

- **Q: What is a WSDL?**
- **A:** Web Services Description Language. It's an XML document that describes the functions of a SOAP service, acting as a formal contract between the provider and the consumer.

- **Q: When would you still choose SOAP over REST?**
- **A:** In enterprise scenarios requiring complex security (WS-Security), ACID transactions, or when strict contracts are needed across different languages and platforms.
