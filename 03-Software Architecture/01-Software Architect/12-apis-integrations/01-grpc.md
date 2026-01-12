---
---

# gRPC (Remote Procedure Calls)

gRPC is a modern, high-performance, open-source RPC framework initially developed by Google. It uses **HTTP/2** for transport, **Protocol Buffers** as the interface description language, and provides features such as authentication, bidirectional streaming, and flow control.

## 1. Definition & Core Concepts

### **Protocol Buffers (Protobuf)**
*   **Definition**: A language-neutral, platform-neutral, extensible mechanism for serializing structured data.
*   **Role**: Acts as both the IDL (Interface Definition Language) and the message exchange format.
*   **Benefit**: Binary serialization is much smaller and faster than text-based formats like JSON or XML.

### **HTTP/2 Transport**
gRPC leverages HTTP/2 features that are not available in HTTP/1.1:
*   **Binary Framing**: Messages are broken into smaller frames, making them easier to parse.
*   **Multiplexing**: Multiple requests/responses can be sent over a single TCP connection simultaneously.
*   **Header Compression (HPACK)**: Reduces overhead by compressing repetitive headers.
*   **Server Push**: Allows the server to send data to the client before it is requested.

### **Interface Definition Language (IDL)**
*   Services are defined in `.proto` files specifying methods and their request/response message types.
*   Code is generated from these files for various languages (Go, Java, Python, etc.), ensuring type safety across services.

## 2. Communication Patterns

| Pattern | Description | Use Case |
| :--- | :--- | :--- |
| **Unary** | Traditional Request/Response. Client sends one request, gets one response. | Standard API calls (e.g., Fetch user by ID). |
| **Server Streaming** | Client sends one request; server returns a stream of messages. | Real-time updates, logs streaming. |
| **Client Streaming** | Client sends a stream of messages; server returns one response. | Uploading large files in chunks. |
| **Bidirectional Streaming** | Both client and server send a sequence of messages using a read-write stream. | Chat applications, real-time gaming. |

## 3. Pros & Cons vs REST

### **Pros**
*   **Performance**: Binary serialization and HTTP/2 make it significantly faster than REST/JSON.
*   **Strict Contract**: Code generation ensures consistency between client and server.
*   **Streaming**: Native support for various streaming patterns.
*   **Language Agnostic**: Easy cross-language communication.

### **Cons**
*   **Human Readability**: Binary messages are hard to debug without specialized tools (e.g., `grpcurl`, `Postman`).
*   **Browser Support**: Browsers don't fully support HTTP/2 trailers, requiring `gRPC-Web` and a proxy (like Envoy).
*   **Ecosystem**: Steeper learning curve compared to simple JSON-over-HTTP.

## 4. Go Implementation

### **`calculator.proto`**
```proto
syntax = "proto3";

package calculator;

// Define where the generated Go code will reside
option go_package = "./pb";

service Calculator {
  // Unary
  rpc Add (AddRequest) returns (AddResponse);
  // Server Streaming
  rpc StreamPrimes (PrimeRequest) returns (stream PrimeResponse);
}

message AddRequest {
  int32 a = 1;
  int32 b = 2;
}

message AddResponse {
  int32 result = 1;
}

message PrimeRequest {
  int32 number = 1;
}

message PrimeResponse {
  int32 factor = 1;
}
```

### **Go Server Implementation**
```go
package main

import (
	"context"
	"log"
	"net"

	"google.golang.org/grpc"
	"pb" // Generated code package
)

type server struct {
	pb.UnimplementedCalculatorServer
}

func (s *server) Add(ctx context.Context, req *pb.AddRequest) (*pb.AddResponse, error) {
	return &pb.AddResponse{Result: req.A + req.B}, nil
}

func (s *server) StreamPrimes(req *pb.PrimeRequest, stream pb.Calculator_StreamPrimesServer) error {
	num := req.Number
	divisor := int32(2)
	for num > 1 {
		if num%divisor == 0 {
			if err := stream.Send(&pb.PrimeResponse{Factor: divisor}); err != nil {
				return err
			}
			num /= divisor
		} else {
			divisor++
		}
	}
	return nil
}

func main() {
	lis, err := net.Listen("tcp", ":50051")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	s := grpc.NewServer()
	pb.RegisterCalculatorServer(s, &server{})
	log.Printf("server listening at %v", lis.Addr())
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```

### **Go Client Implementation**
```go
package main

import (
	"context"
	"io"
	"log"
	"time"

	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
	"pb"
)

func main() {
	conn, err := grpc.Dial("localhost:50051", grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()
	c := pb.NewCalculatorClient(conn)

	// Unary Call
	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()
	r, _ := c.Add(ctx, &pb.AddRequest{A: 10, B: 20})
	log.Printf("Sum: %d", r.GetResult())

	// Server Streaming Call
	stream, _ := c.StreamPrimes(context.Background(), &pb.PrimeRequest{Number: 120})
	for {
		res, err := stream.Recv()
		if err == io.EOF {
			break
		}
		log.Printf("Prime Factor: %d", res.GetFactor())
	}
}
```

## 5. Interview Questions

**Q1: What is the main difference between gRPC and REST?**
*   **Answer**: REST is usually based on HTTP/1.1 and uses JSON (text) for data exchange, while gRPC uses HTTP/2 and Protocol Buffers (binary). gRPC is typically faster, supports streaming natively, and uses a strict contract (IDL), whereas REST is more flexible and human-readable.

**Q2: Why does gRPC use HTTP/2 instead of HTTP/1.1?**
*   **Answer**: HTTP/2 provides multiplexing (multiple requests over one connection), header compression (HPACK), and binary framing. These features reduce latency, save bandwidth, and allow for efficient bidirectional streaming which is a core feature of gRPC.

**Q3: What are Protocol Buffers and why are they used?**
*   **Answer**: Protocol Buffers are a binary serialization format. They are used because they are more compact and faster to serialize/deserialize than JSON. They also provide a language-neutral IDL that allows for automatic code generation, ensuring type safety and consistency across different microservices.

**Q4: Explain the four types of gRPC service methods.**
*   **Answer**: 
    1.  **Unary**: Single request, single response.
    2.  **Server Streaming**: Single request, stream of responses.
    3.  **Client Streaming**: Stream of requests, single response.
    4.  **Bidirectional Streaming**: Stream of requests, stream of responses.

**Q5: How do you handle cross-cutting concerns like Authentication or Logging in gRPC?**
*   **Answer**: gRPC uses **Interceptors** (similar to middleware in web frameworks). You can define client-side and server-side interceptors to inspect and modify requests/responses, manage authentication tokens in metadata, or log RPC details without cluttering the business logic.
