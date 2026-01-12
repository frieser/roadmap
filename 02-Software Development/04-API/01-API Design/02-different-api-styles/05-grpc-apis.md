# gRPC APIs

## Summary
gRPC (Google Remote Procedure Call) is a modern, high-performance, open-source RPC framework that can run in any environment. It uses **HTTP/2** for transport and **Protocol Buffers** (Protobuf) as the interface description language. It enables client and server applications to communicate transparently, making it easier to build connected systems with features like bidirectional streaming and efficient binary serialization.

## Detailed Explanation

### 1. What is gRPC?
gRPC is a framework that implements the **RPC (Remote Procedure Call)** model. Unlike REST, which is resource-oriented, RPC is procedure-oriented, meaning you call functions on a remote server as if they were local functions.

### 2. Protocol Buffers (Protobuf) vs JSON
Protobuf is the default serialization mechanism for gRPC.

| Feature | Protocol Buffers | JSON |
| --- | --- | --- |
| **Format** | Binary (Compact) | Text (Verbose) |
| **Speed** | Very Fast (Optimized for CPU) | Slower (Parsing overhead) |
| **Schema** | Required (`.proto` file) | Optional |
| **Type Safety** | Strongly Typed | Weakly Typed |
| **Browser Support**| Limited (needs proxy) | Native |

### 3. HTTP/2 and Bidirectional Streaming
gRPC is built on **HTTP/2**, which provides several advantages over HTTP/1.1 used by traditional REST:
- **Multiplexing:** Allows multiple requests and responses over a single TCP connection.
- **Binary Framing:** More efficient parsing.
- **Header Compression:** Reduces overhead using HPACK.
- **Bidirectional Streaming:** Both client and server can send a sequence of messages using a single persistent connection.

### 4. gRPC in Go

#### Step 1: Define the Service (.proto)
Create a file named `greet.proto`:
```proto
syntax = "proto3";

option go_package = "./pb";

service GreetService {
  // Unary RPC
  rpc SayHello (HelloRequest) returns (HelloResponse);
  
  // Bidirectional Streaming RPC
  rpc Chat (stream ChatMessage) returns (stream ChatMessage);
}

message HelloRequest { string name = 1; }
message HelloResponse { string message = 1; }
message ChatMessage { string user = 1; string text = 2; }
```

#### Step 2: Generate Go Code
Use the `protoc` compiler:
```bash
protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    pb/greet.proto
```

#### Step 3: Implement the Server
Evidence ([grpc-go/examples](https://github.com/grpc/grpc-go/blob/master/examples/helloworld/greeter_server/main.go)):
```go
package main

import (
	"context"
	"log"
	"net"

	"google.golang.org/grpc"
	pb "your-project/pb"
)

type server struct {
	pb.UnimplementedGreetServiceServer
}

func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloResponse, error) {
	return &pb.HelloResponse{Message: "Hello " + in.GetName()}, nil
}

func main() {
	lis, err := net.Listen("tcp", ":50051")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	s := grpc.NewServer()
	pb.RegisterGreetServiceServer(s, &server{})
	log.Printf("server listening at %v", lis.Addr())
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```

## Interview Questions

*   **Q: What are the main benefits of using gRPC over REST?**
    *   **A:** gRPC offers better performance due to Protobuf binary serialization and HTTP/2 multiplexing. It provides strict type safety through schemas and supports bidirectional streaming, which is difficult in traditional REST.
*   **Q: How does HTTP/2 improve gRPC performance?**
    *   **A:** It allows multiple concurrent streams over a single connection (multiplexing), reduces latency through header compression, and uses binary framing instead of text, making it faster to parse.
*   **Q: What is the difference between Unary, Client Streaming, Server Streaming, and Bidirectional Streaming?**
    *   **A:** 
        *   **Unary:** Single request, single response.
        *   **Server Streaming:** Single request, stream of responses.
        *   **Client Streaming:** Stream of requests, single response.
        *   **Bidirectional:** Stream of requests and responses simultaneously.
*   **Q: Why is Protobuf faster than JSON?**
    *   **A:** Protobuf is a binary format that skips the expensive string parsing required by JSON. It also uses field tags (numbers) instead of field names, significantly reducing the payload size.
*   **Q: How do you handle errors in gRPC?**
    *   **A:** gRPC uses a set of standard status codes (e.g., `OK`, `NOT_FOUND`, `INTERNAL`). You can also attach "error details" to the status to provide more context to the client.
