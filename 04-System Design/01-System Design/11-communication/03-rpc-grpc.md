---
---

## Summary
**RPC (Remote Procedure Call)** allows a program to execute a procedure on a different address space (usually another server) as if it were a local call. **gRPC** is a modern, high-performance RPC framework developed by Google that uses **Protocol Buffers (Protobuf)** for serialization and **HTTP/2** for transport, enabling features like multiplexing and streaming.

## Detailed Explanation

### 1. RPC Basics
RPC abstracts the network communication, making distributed systems look like local monolithic systems.
*   **Stubs**: Client-side and server-side proxies that handle serialization (marshaling) and network transmission.
*   **Workflow**: Client calls a stub → Stub serializes data → Data sent over network → Server stub deserializes → Server executes function → Result returned.

### 2. gRPC (Google RPC)
gRPC improves upon traditional RPC by using modern technologies:
*   **Protocol Buffers (Protobuf)**: A binary serialization format that is smaller and faster than JSON/XML. It requires a `.proto` definition file.
*   **HTTP/2**: Enables long-lived connections, header compression, and bi-directional streaming.
*   **Service Types**:
    1.  **Unary**: One request, one response.
    2.  **Server Streaming**: One request, many responses.
    3.  **Client Streaming**: Many requests, one response.
    4.  **Bi-directional Streaming**: Many requests, many responses.

### Go Context: `grpc-go`
Go is a first-class citizen for gRPC. The `google.golang.org/grpc` package is the standard implementation.

#### Protobuf Definition (`user.proto`)
```proto
syntax = "proto3";
package user;
option go_package = "./pb";

service UserService {
  rpc GetUser (UserRequest) returns (UserResponse) {}
}

message UserRequest {
  string id = 1;
}

message UserResponse {
  string name = 1;
  int32 age = 2;
}
```

#### gRPC Server in Go
```go
package main

import (
	"context"
	"net"
	"google.golang.org/grpc"
	"myproject/pb"
)

type server struct {
	pb.UnimplementedUserServiceServer
}

func (s *server) GetUser(ctx context.Context, req *pb.UserRequest) (*pb.UserResponse, error) {
	return &pb.UserResponse{Name: "Bob", Age: 30}, nil
}

func main() {
	lis, _ := net.Listen("tcp", ":50051")
	s := grpc.NewServer()
	pb.RegisterUserServiceServer(s, &server{})
	s.Serve(lis)
}
```

## Interview Questions
*   **Q: Why is gRPC faster than REST/JSON?**
    *   **A:** gRPC uses Protobuf (binary format) which is much smaller and faster to serialize/deserialize than JSON (text format). It also uses HTTP/2, which supports multiplexing and reduces connection overhead.
*   **Q: What is the purpose of the `.proto` file in gRPC?**
    *   **A:** The `.proto` file serves as the "Source of Truth" for the API contract. It defines the service methods and message structures in a language-neutral way, which is then used to generate code for clients and servers in various languages.
*   **Q: When should you use gRPC over REST?**
    *   **A:** gRPC is ideal for internal microservices communication where performance is critical, when you need low-latency streaming, or when you want strict contract enforcement via strong typing.
