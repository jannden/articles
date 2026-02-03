# Golang server with gRPC and REST API at the same time

_Originally published on Jul 19, 2023 [here](https://enlear.academy/golang-server-with-grpc-and-rest-api-at-the-same-time-66547d547969)._

---

This article explains a simple way to create a Go server that listens to gRPC and REST requests at the same time.

The best part is that we will use a simple Protocol Buffer schema that will be compiled for both types of APIs.

TL;DR --- You can see it done in my [Github repository](https://github.com/jannden/golang-examples/tree/main/grpc-with-rest).

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*HWHRX26D1ZStK1vwcJLrIA.png)

Let's start by initializing a new Go project.

```react
$ mkdir hello
$ cd hello
$ go mod init github.com/your-username/hello
```

### Create Protocol Buffers

### Install Protocol Buffer Compiler

If you don't have `protoc` installed yet, please follow [this installation guide](https://grpc.io/docs/protoc-installation/).

Then add `protoc` support for [grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway) and optionally for [open-api-v2](https://swagger.io/specification/v2/):

```react
$ go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway@latest
$ go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2@latest
```

Next, we need to ensure that the required dependencies are available to the compiler at compile time. This can be done by manually cloning and copying the relevant files from the [googleapis repository](https://github.com/googleapis/googleapis) into your own project in the folder `./google/api`. The files you will need are:

```react
google/api/annotations.proto
google/api/field_behavior.proto
google/api/http.proto
google/api/httpbody.proto
```

### Create proto file

Now we will create the Protocol Buffers. Create a file `./proto/hello.proto` and fill it with this code:

```react
syntax = "proto3";
```

```react
package hello;option go_package = "github.com/your-username/hello";import "google/api/annotations.proto";option (grpc.gateway.protoc_gen_swagger.options.openapiv2_swagger) = {
 info: {
  title: "HelloService";
  version: "1.0";
  contact: {
   name: "grpc-with-rest";
   url: "https://github.com/your-username/hello";
    };
  };
  schemes: HTTP;
  consumes: "application/json";
  produces: "application/json";
  responses: {
  key: "404";
  value: {
   description: "Returned when the resource does not exist.";
   schema: {
    json_schema: {
     type: STRING;
    }
   }
  }
 }
};message HelloRequest {
 string name = 1;
}message HelloResponse {
 string message = 1;
}service HelloService {
 rpc SayHello (HelloRequest) returns (HelloResponse) {
  option(google.api.http) = {
   get: "/api/hello/{name}",
  };
 }
}
```

### Compile the protos

Last but not least, let's run a `protoc` script in your console that will compile the Protocol Buffers into the necessary code:

```react
$ protoc -I ./proto\
  --go_out ./proto --go_opt paths=source_relative\
  --go-grpc_out ./proto --go-grpc_opt paths=source_relative\
  --grpc-gateway_out ./proto --grpc-gateway_opt paths=source_relative\
  --openapiv2_out ./proto --openapiv2_opt use_go_templates=true\
  ./proto/hello.proto
```

This should create 4 new files in your `./proto` folder.

### Create the server

Create a file `./server/main.go` and fill it with this code:

```react
package main

import (
 "context"
 "log"
 "net"
 "net/http"

 "github.com/grpc-ecosystem/grpc-gateway/v2/runtime"
 "google.golang.org/grpc"
 "google.golang.org/grpc/credentials/insecure"
 "google.golang.org/grpc/reflection"

 pb "github.com/jannden/golang-examples/grpc-with-rest/proto"
)

type server struct {
 pb.UnimplementedHelloServiceServer
}

func NewServer() *server {
 return &server{}
}

func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloResponse, error) {
 return &pb.HelloResponse{Message: "Hello, " + in.Name}, nil
}

func main() {
 lis, err := net.Listen("tcp", ":50051")
 if err != nil {
  log.Fatalln("Failed to listen:", err)
 }

 s := grpc.NewServer()
 reflection.Register(s)
 pb.RegisterHelloServiceServer(s, &server{})

 go func() {
  log.Println("Serving gRPC on 0.0.0.0:50051")
  if err := s.Serve(lis); err != nil {
   log.Fatalf("Failed to serve gRPC server: %v", err)
  }
 }()

 conn, err := grpc.DialContext(
  context.Background(),
  "0.0.0.0:50051",
  grpc.WithBlock(),
  grpc.WithTransportCredentials(insecure.NewCredentials()),
 )
 if err != nil {
  log.Fatalln("Failed to dial server:", err)
 }

 gwmux := runtime.NewServeMux()
 err = pb.RegisterHelloServiceHandler(context.Background(), gwmux, conn)
 if err != nil {
  log.Fatalln("Failed to register gateway:", err)
 }

 gwServer := &http.Server{
  Addr:    ":50052",
  Handler: gwmux,
 }

 log.Println("Serving gRPC-Gateway for REST on http://0.0.0.0:50052")
 if err := gwServer.ListenAndServe(); err != nil {
  log.Fatalf("Failed to serve gRPC-Gateway server: %v", err)
 }
}
```

### Run the server

Run the server:

```react
$ go run server/main.go
```

Open a new terminal.

Test the connection with gRPC using [grpcurl](https://github.com/fullstorydev/grpcurl):

```react
$ grpcurl -d '{"name":"John"}' -plaintext 0.0.0.0:50051 hello.HelloService/SayHello
```

Test the connection with REST with curl:

```react
$ curl 0.0.0.0:50052/api/hello/John
```

### Conclusion

Hope you got a good grasp on creating a Go server with gRPC and REST 😊

If you found it helpful, please click the clap 👏 button.
And feel free to comment! I'd be happy to help :)
