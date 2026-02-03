## Golang Crash Course --- with gRPC and Postgres

_Originally published on Jul 1, 2023 [here](https://enlear.academy/golang-crash-course-2023-with-grpc-and-postgres-82248e548683)._

---

I will show you how to create a gRPC server with Postgres integration and protobufs in Go.

TL;DR --- You can see the final code in my [Github repository](https://github.com/jannden/golang-examples/tree/main/grpc-postgres-migrate-gorm).

Let's get started.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*phrWFY9AkrgfzzO8YurmEg.png)

### 1\. Install Go

Download the latest version from [the official Go website](https://golang.org/dl/).

### 2\. Setup your workspace

Go requires you to set up your workspace before you start writing code. You need to create a directory where you will keep your Go code and set some environment variables. Here's an example for iOS (if you are using another OS, read the official docs for related installation instructions):

```react
$ mkdir $HOME/go
$ export GOPATH=$HOME/go
$ export PATH=$PATH:$GOPATH/bin
```

### 3\. Write your first program

Create a new directory for your project:

```react
$ mkdir hello
$ cd hello
```

Initialize the project using the `go mod init` command, which will create a new `go.mod` file in your project directory:

```react
$ go mod init github.com/your-username/hello
```

Replace `github.com/your-username/hello` with the appropriate module path for your project.

Once your workspace is set up, you can start writing your first program. Open a text editor and create a file called `hello.go`. In this file, write the following code:

```react
package main
import "fmt"
func main() {
 fmt.Println("Hello, World!")
}
```

This is a simple program written in Go that prints the text "Hello, World!" to the console.

The `package main` line indicates that this program is meant to be compiled as an executable rather than a library.

The `import "fmt"` line imports the "fmt" package, which contains functions for formatting and printing text.

The `func main()` line defines the program's entry point. Within this function, the `fmt.Println()` function is called with the argument "Hello, World!", which causes that text to be printed to the console.

### 4\. Run your program

To run your program, open a terminal window, navigate to the directory where your `hello.go` file is located, and run the following command:

```react
go run hello.go
```

You should see the output '*Hello, World!'* printed to the console.

### A couple of words about gRPC, REST, and GraphQL

An API is like a waiter in a restaurant. The waiter takes your order and tells the kitchen what you want to eat. In the same way, an API takes requests from a computer program and sends them to a server to get the data or service you want. It's like a messenger between different computer programs, making them work together.

There are various ways to implement an API. Three popular methods are gRPC, REST, and GraphQL. Each of these methods has its own advantages and disadvantages, so let's compare them to help you decide which one is best suited for your project.

gRPC is a high-performance, open-source, universal RPC framework that uses protocol buffers as its data format. It is designed to be faster and more efficient than traditional APIs, and it works by creating a client and a server. The client sends a request to the server, and the server returns a response. The communication between the client and server is done over HTTP/2, making it faster than REST. gRPC also supports many programming languages, making it easy to use in different environments.

REST is the most used web standard for APIs. It is based on HTTP/1.2. It is easier to understand and implement than gRPC. REST works by defining a set of operations that can be performed on resources, such as GET, POST, PUT, and DELETE. REST is also scalable and easy to maintain, making it a popular choice for building web applications.

GraphQL allows clients to define the structure of the data they require, making it more efficient than REST for complex queries. GraphQL also provides a type system, making it easier to validate data and write more predictable code. GraphQL also supports subscriptions, allowing clients to receive real-time updates when data changes.

So, which one should you use? It depends on your project requirements. If you need a high-performance API and have complex data models, gRPC may be the best choice. If you need a simple and flexible API that is easy to implement and understand, REST may be the best choice. If you have a complex data model and need more efficient queries, GraphQL may be the best choice.

We will proceed with gRPC which is arguably the most used Go implementation of APIs in corporate settings. In order to start with gRPC, we need to be able to use protocol buffers first.

### Protocol buffers

Protocol buffers are a way to format and transmit data between different computer programs. They use a binary format, which means the data is represented in a way that is easy for computers to read and process quickly. Think of it like a special language that different programs can use to communicate with each other.

In order to use protocol buffers, we need to install a compiler called `protoc`. Please follow the official [Protocol Buffer Compiler Installation](https://grpc.io/docs/protoc-installation/) guide.

To verify you have successfully installed protoc, run the following command in your console:

```react
$ protoc --version
```

As a next step, we need to add Go plugins for the `protoc` :

```react
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2
```

Next, we need to tell our CLI where to find our Go installation.

If on Mac, run:

```react
$ open ~/.zshrc
```

In the opened file, add these two lines:

```react
export GO_PATH=~/go
export PATH=$PATH:/$GO_PATH/bin
```

### Setting up the project

Let's create the working directory now:

```react
$ mkdir go-grpc-postgres
```

Move into the directory:

```react
$ cd go-grpc-postgres/
```

Create a go module:

```react
$ go mod init example.com/go-grpc-postgres
```

Create folders for server, client, and proto files:

```react
$ mkdir server client proto
```

### Create the proto

Create a new file in the `proto` folder called `todo.proto`, so the path will look like this: `./proto/todo.proto`. Now fill it with this code:

```react
syntax="proto3";

package todo;

option go_package = "example.com/go-grpc-postgres/proto/todo";

message NewTodo {
  string name = 1;
  string description = 2;
  bool done = 3;
}

message Todo {
  string id = 1;
  string name = 2;
  string description = 3;
  bool done = 4;
}

service TodoService {
  rpc CreateTodo (NewTodo) returns (Todo);
}
```

This defines a schema for ourTodo API. The schema includes two message types, NewTodo and Todo. Additionally, there is a service called TodoService, which includes one method called CreateTodo. This method takes a NewTodo message as input and returns a complete Todo message as output. In other words, the CreateTodo method allows users to create a new to-do task and returns the created task with an ID.

### Compile the proto

Create a new file in the parent directory `./compile-protos.sh` and fill it with this command:

```react
protoc\
  --go_out=.\
  --go_opt=paths=source_relative\
  --go-grpc_out=.\
  --go-grpc_opt=paths=source_relative\
  proto/todo.proto
```

This command uses the *protoc* compiler to generate Go code for our Protocol Buffers schema file `todo.proto`. It will place the generated files in the current directory, with import paths relative to the source directory. The generated Go code can then be used to implement a Todo API in Go.

Make the command file executable:

```react
$ chmod +x compile-protos.sh
```

And now run it --- so just basically write the relative path and the file name in the console:

```react
$ ./compile-protos.sh
```

This will compile the Protocol Buffers in the Go code.

### Implementing gRPC server and client in Go

Since we have the proto ready, we can move ahead with Golang.

First, let's install gRPC support for our Go project:

```react
$ go get google.golang.org/grpc
```

### Create the server

Create file `./server/main.go` and fill it with this code, which implements a gRPC server for a Todo API listening for requests on port 50051 and responding with a `pb.Todo` message when a new todo is created:

```react
package main

import (
 "context"
 "log"
 "net"

 pb "example.com/go-grpc-postgres/proto"

 "google.golang.org/grpc"
)

type TodoServer struct {
 pb.UnimplementedTodoServiceServer
}

func (s *TodoServer) CreateTodo(ctx context.Context, req *pb.NewTodo) (*pb.Todo, error) {
 log.Printf("received: %v", req.GetName())
 todo := &pb.Todo{
  Name:        req.GetName(),
  Description: req.GetDescription(),
 }

 return todo, nil
}

func main() {
 lis, err := net.Listen("tcp", ":50051")
 if err != nil {
  log.Fatalf("tcp connection failed: %v", err)
 }
 log.Printf("listening at %v", lis.Addr())

 s := grpc.NewServer()

 pb.RegisterTodoServiceServer(s, &TodoServer{})
 if err := s.Serve(lis); err != nil {
  log.Fatalf("grpc server failed: %v", err)
 }
}
```

We define a struct called `TodoServer`, which implements the `pb.UnimplementedTodoServiceServer` interface. That provides a default implementation for all of the services, which means that if you add a new service to the `proto` file and then recompile your application it will compile without error and return an unimplemented error should a client attempt to call the new service.

We add the `CreateTodo` method to the `TodoServer` which logs the received request and creates a new `pb.Todo` message with the `Name` and `Description` fields from the `pb.NewTodo` message. In the end, it returns the created `pb.Todo` message.

In the `main` function, we prepare a listener for TCP connections on port 50051 with `net.Listen()`. Then we create a new gRPC server using `grpc.NewServer()`. The `pb.RegisterTodoServiceServer` method is used to register the `TodoServer` with the gRPC server. Finally, the gRPC server is started by connecting the listener to the server.

### Create the client

Create file `./client/main.go` and fill it with this code, which creates new todo items using a gRPC client for a Todo API:

```react
package main

import (
 "context"
 "log"
 "time"

 pb "example.com/go-grpc-postgres/proto"

 "google.golang.org/grpc"
 "google.golang.org/grpc/credentials/insecure"
)

func main() {
   conn, err := grpc.Dial(":50051", grpc.WithTransportCredentials(insecure.NewCredentials()))
   if err != nil {
       log.Fatalf("problem with the server: %v", err)
   }

   defer conn.Close()

   c := pb.NewTodoServiceClient(conn)

   ctx, cancel := context.WithTimeout(context.Background(), time.Second)

   defer cancel()

   newTodos := []*pb.NewTodo{
       {Name: "Buy pizza", Description: "Make sure to get a spicy one.", Done: false},
       {Name: "Have fun", Description: "Eat and enjoy.", Done: false},
   }

   for _, newTodo := range newTodos {
       res, err := c.CreateTodo(ctx, &pb.NewTodo{Name: newTodo.Name, Description: newTodo.Description, Done: newTodo.Done})
       if err != nil {
           log.Fatalf("could not create todo: %v", err)
       }

       log.Printf(`
           ID: %s
           Name: %s
           Description: %s
           Done: %v
       `, res.GetId(), res.GetName(), res.GetDescription(), res.GetDone())
   }
}
```

We create a gRPC client connection using `grpc.Dial` to connect to the gRPC server listening on port 50051. Then we create `pb.NewTodoServiceClient` using the connection and a context object with a timeout of one second. Two new todo items are created and stored in a slice of `pb.NewTodo` structs.

In a simple `for` loop, we iterate over the new to-do items and call the `CreateTodo` method on the gRPC client with the to-do item data. The response from the gRPC server is logged to the console.

### Let the client contact the server

Now we have everything ready to try this out.

Run the server:

```react
$ go run server/main.go
```

Open a new terminal and run the client:

```react
$ go run client/main.go
```

You should see an output in both terminals.

### Add the database

A real project isn't completed without saving the data. So we will focus on connecting to a database now.

To simulate a corporate setting, we will set up database migrations as well. Database migrations are a way to manage changes to a database schema over time by versioning and applying SQL scripts to create, modify or delete database objects.

### Create Postgres database

Use Render.com to create a Postgres database for free. It's a pretty straightforward process.

After creating a PostgreSQL database on Render.com, note down the External Database URL. It's located on the External Connection tab under the Connect dropdown in the top right corner:

![](https://miro.medium.com/v2/resize:fit:630/1*Qx7zvDTKrg2onTjJHVZDeA.png)

### Install Golang migrate package

Install [golang-migrate](https://github.com/golang-migrate/migrate) in order to start with DB migrations.

If you are using iOS and have homebrew installed, you can install golang-migrate it with this command (otherwise read their docs for other installation options):

```react
$ brew install golang-migrate
```

Database migrations are a way to manage changes to a database schema over time. They are used to keep the database schema in sync with the application code that interacts with it.

In the context of the golang-migrate module, database migrations involve writing SQL scripts to create, modify or delete database objects. These SQL scripts are versioned and stored in the filesystem, and the golang-migrate tool is used to apply them to the database in a specified order.

Let's create a migration for database creation:

```react
$ migrate create -ext sql -dir migrations -seq create_todos_table
```

If there were no errors, we should have two files available in the `./migrations` folder:

- 000001_create_users_table.down.sql
- 000001_create_users_table.up.sql

In the `.up.sql` file let's create the table with this SQL script:

```react
CREATE TABLE todos (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    description TEXT,
    done BOOLEAN NOT NULL DEFAULT false
);
```

And in the `.down.sql` let's add the ability to revert the up change by having this option to delete the table:

```react
DROP TABLE IF EXISTS todos;
```

When using Migrate CLI we need to pass the database URL. Let's export it to a variable for convenience (update the URL in the code snippet bellow with the URL you got from Render.com):

```react
$ export DATABASE_URL='postgres://user_name:password@localhost:5432/database_name?sslmode=disable'
```

Now you should be ready to run the migration:

```react
$ migrate -database ${DATABASE_URL} -path migrations up
```

The database table should now be created.

### Connect the server to the database

Since we have the database ready now, let's connect our server to it.

Install additional Go modules so that we are easily able to communicate with the database:

```react
$ go get gorm.io/driver/postgres
$ go get gorm.io/gorm
```

Add those packages to the imports in `./server/main.go` along with the `os` package. So the imports will look like this:

```react
import (
 "context"
 "log"
 "net"
 "os"

 pb "example.com/go-grpc-postgres/proto"

 "google.golang.org/grpc"
 "gorm.io/driver/postgres"
 "gorm.io/gorm"
)

// ... rest of the code
```

Add two new functions to `./server/main.go`. Now we will be able to connect to the database.

```react
// ...

func init() {
 DatabaseConnection()
}
var DB *gorm.DB
var err error

func DatabaseConnection() {
 DB, err = gorm.Open(postgres.Open(os.Getenv("DATABASE_URL")), &gorm.Config{})
 if err != nil {
   log.Fatal("db connection error: ", err)
 }
 log.Println("db connection successful")
}

// ...
```

The `init()` function initializes the database connection by calling the `DatabaseConnection()` function when the program starts.

The `DatabaseConnection()` function uses the GORM library to connect to a PostgreSQL database specified by the `DATABASE_URL` environment variable that we exported earlier. If the connection is successful, a global `DB` variable is set to the database object, which can be used to perform database operations. If the connection fails, the function logs an error and exits the program.

And now, with the DB connection ready, we can save the data. So we will update the `CreateTodo()` method of the `TodoServer` like this:

```react
func (s *TodoServer) CreateTodo(ctx context.Context, req *pb.NewTodo) (*pb.Todo, error) {
 log.Printf("Received: %v", req.GetName())
 todo := &pb.Todo{
  Name:        req.GetName(),
  Description: req.GetDescription(),
 }
 res := DB.Create(&todo)
 if res.RowsAffected == 0 {
   return nil, errors.New("error saving todo")
 }
 return &pb.Todo{
     Id:          todo.Id,
     Name:        todo.Name,
     Description: todo.Description,
     Done:        false,
 }, nil
}
```

In the method, we first define `todo` and set properties `Name` and `Description`. We then save it to the database with `DB.Create(&todo)` .

`Create()` is a method from the gorm library, which in our case adds the Postgres generated `Id` property to `todo` automatically on a successful save. That's why we are able to access `todo.Id` in the `return` statement at the end.

### Updating the client

In order to display the newly generated `Id`, we need to convert it from int64 to string. Add package `strconv` to the imports in `./client/main.go`. So the imports will look like this:

```react
import (
 "context"
 "log"
 "strconv"
 "time"

 pb "example.com/go-grpc-postgres/proto"

 "google.golang.org/grpc"
 "google.golang.org/grpc/credentials/insecure"
)

// ...
```

Now, at the end of the `main()` function in `./client/main.go`, convert the `res.getId()` from int64 to string with statement `strconv.FormatInt(res.GetId(), 10)`. So the whole log will look like this:

```react
// ...

 log.Printf(`
     ID: %s
     Name: %s
     Description: %s
     Done: %v
    `,
    strconv.FormatInt(res.GetId(), 10),
    res.GetName(),
    res.GetDescription(),
    res.GetDone()
  )

// ...
```

And that's it! We have a working example of a To-Do app with Golang, GRPC, and Postgres.

As an exercise, try to implement a new method with which you will be able to set the property `Done` to `true`.

You can find the whole finished code in my repo.

### Conclusion

Hope you got a good grasp on using gRPC in Go 😊

If you found it helpful, please click the clap 👏 button.
And feel free to comment! I'd be happy to help :)
