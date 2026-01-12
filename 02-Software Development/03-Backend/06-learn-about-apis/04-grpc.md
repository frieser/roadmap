---
---

## Summary
gRPC (Google Remote Procedure Call) is a modern open-source, high-performance RPC framework. It uses Protocol Buffers (Protobuf) as its Interface Definition Language (IDL) and runs over HTTP/2, enabling features like multiplexing, streaming, and header compression.

## Detailed Explanation
Developed by Google, gRPC is designed for efficient communication between microservices.

### Protocol Buffers (Protobuf)
Instead of JSON or XML, gRPC uses binary serialization called Protobuf. You define your data structures and service methods in a `.proto` file, and gRPC generates the client and server code for multiple languages.

### Types of gRPC
1. **Unary**: Traditional request-response.
2. **Server Streaming**: Client sends one request, server sends a stream of responses.
3. **Client Streaming**: Client sends a stream of requests, server sends one response.
4. **Bidirectional Streaming**: Both sides send a stream of messages simultaneously.

### Advantages over REST
- **Performance**: Binary format is much smaller and faster to parse than JSON.
- **HTTP/2**: Multiplexing allows multiple requests over a single TCP connection.
- **Type Safety**: Strong typing enforced by Protobuf.
- **Code Generation**: First-class support for many languages.

## Go Context
Go is one of the primary languages for gRPC.

### Example: Protobuf Definition (`user.proto`)
```protobuf
syntax = "proto3";
package user;
option go_package = "./pb";

message UserRequest {
  string id = 1;
}

message UserResponse {
  string name = 1;
  int32 age = 2;
}

service UserService {
  rpc GetUser(UserRequest) returns (UserResponse);
}
```

### Example: Go Server Implementation
```go
package main

import (
	"context"
	"log"
	"net"
	"google.golang.org/grpc"
	"pb" // generated code
)

type server struct {
	pb.UnimplementedUserServiceServer
}

func (s *server) GetUser(ctx context.Context, req *pb.UserRequest) (*pb.UserResponse, error) {
	return &pb.UserResponse{Name: "John Doe", Age: 30}, nil
}

func main() {
	lis, _ := net.Listen("tcp", ":50051")
	s := grpc.NewServer()
	pb.RegisterUserServiceServer(s, &server{})
	log.Println("Server listening on :50051")
	s.Serve(lis)
}
```

## Interview Questions
- **Q: What is the role of HTTP/2 in gRPC?**
- **A:** HTTP/2 provides the underlying transport for gRPC. It enables features like bidirectional streaming, multiplexing (sending multiple requests over one connection), and binary framing, which improves performance.

- **Q: Why is gRPC faster than REST/JSON?**
- **A:** It uses a binary serialization format (Protobuf) which is much smaller than text-based JSON. It also benefits from HTTP/2's efficiency and removes the overhead of repeated text headers.

- **Q: What is a Protobuf file?**
- **A:** A `.proto` file is a schema definition where you define the structure of your data and the methods exposed by your service. It acts as a single source of truth for generating code in various languages.
