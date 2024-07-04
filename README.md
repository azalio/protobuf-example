#grpc #learning 

**Protobuf** (Protocol Buffers) — это формат сериализации данных, разработанный компанией Google. 

Он эффективно и компактно хранит структурированные данные в двоичной форме, что позволяет быстрее передавать их по сети. 

Protobuf поддерживает широкий спектр языков программирования и является платформонезависимым, что означает, что программы, написанные с его использованием, могут быть легко перенесены на другие платформы.

Этот формат используется в различных приложениях, таких как веб-сервисы, базы данных, RPC-системы и форматы файлов. Он поддерживает множество типов данных, включая строки, целые числа, плавающие числа, булевы, перечисления, карты и многое другое.

- Наша цель написать простейшее клиент-серверное приложение на golang с использованием google protobuf
- Ставим необходимые инструменты

```bash
   go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
   go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

- Формируем необходимые каталоги и файлы (go.mod сделаем потом)
```bash

   myservice/
   ├── proto/
   │   └── service.proto
   ├── server/
   │   └── main.go
   ├── client/
   │   └── main.go
   └── go.mod

```

- Определяем схему (proto/service.proto)
```protobuf
   syntax = "proto3"; // определили версию, это важно, потому что у разных версий отличаются опции и синтаксис

   option go_package = "proto"; // Эта опция определяет путь пакета Go, который будет использоваться при генерации кода Go из этого proto-файла. В данном случае, сгенерированный код будет находиться в пакете "proto/".

   package myservice; // Эта строка определяет пространство имен protobuf для этого файла. Это помогает избежать конфликтов имен между различными проектами.

   service Greeter { // Здесь начинается определение сервиса gRPC. Сервис называется "Greeter".
     rpc SayHello (HelloRequest) returns (HelloReply) {} // Это определение метода RPC (Remote Procedure Call) в сервисе. Метод называется "SayHello", принимает запрос типа "HelloRequest" и возвращает ответ типа "HelloReply".
   }

   message HelloRequest { // Начало определения структуры сообщения для запроса.
     string name = 1; // Определение поля в структуре HelloRequest. Это поле типа string с именем "name". Число 1 - это уникальный идентификатор поля в protobuf.
   }

   message HelloReply { // Начало определения структуры сообщения для ответа.
     string message = 1; // Определение поля в структуре HelloReply. Это поле типа string с именем "message". Опять же, 1 - это уникальный идентификатор поля.
   }
   
```

- генерация кода из прото файла
```
protoc --go_out=. --go-grpc_out=. proto/service.proto
```

- появятся дополнительно 2 файла
```
ls -la ./proto
total 32
drwxr-xr-x  5 azalio  593637566   160 Jul  4 12:52 .
drwxr-xr-x  7 azalio  593637566   224 Jul  4 12:20 ..
-rw-r--r--  1 azalio  593637566  6577 Jul  4 12:12 service.pb.go
-rw-r--r--  1 azalio  593637566   285 Jul  4 12:09 service.proto
-rw-r--r--  1 azalio  593637566  3640 Jul  4 12:12 service_grpc.pb.go
```

`proto/service.pb.go`

  Этот файл содержит Go код, сгенерированный из вашего `service.proto` файла. Он включает в себя:

   - Определения структур для сообщений (messages)

```go
type HelloRequest struct {
        state         protoimpl.MessageState
        sizeCache     protoimpl.SizeCache
        unknownFields protoimpl.UnknownFields

        Name string `protobuf:"bytes,1,opt,name=name,proto3" json:"name,omitempty"`
}

type HelloReply struct {
        state         protoimpl.MessageState
        sizeCache     protoimpl.SizeCache
        unknownFields protoimpl.UnknownFields

        Message string `protobuf:"bytes,1,opt,name=message,proto3" json:"message,omitempty"`
}
```

   - Методы сериализации и десериализации для этих структур (показал только для `HelloRequest`)

```go
go
type HelloRequest struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Name string protobuf:"bytes,1,opt,name=name,proto3" json:"name,omitempty"
}

// Сбрасывает состояние сообщения. Это полезно для повторного использования объекта.
func (x *HelloRequest) Reset() {
    *x = HelloRequest{}
    if protoimpl.UnsafeEnabled {
        mi := &file_proto_service_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// Возвращает строковое представление сообщения.
func (x *HelloRequest) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// Метод ProtoMessage — это типовой индикатор (маркер) в сгенерированных структурах Protobuf.
// 1. **Маркер типа**: 
//    - Метод ProtoMessage служит как индикатор для библиотек и инструментов работы с Protobuf, указывая, что данная структура соответствует спецификации Protobuf сообщения. Это позволяет библиотекам выполнять различные операции над этими структурами, такие как сериализация, десериализация, проверка типов и другие.
// 2. **Интерфейсные требования**:
// - В Go метод ProtoMessage используется для проверки соответствия структуры некоторым интерфейсам. Например, всякий раз, когда вам нужно убедиться, что структура — это допустимое Protobuf-сообщение, можно проверить наличие этого метода.
// - Protobuf библиотеки могут полагаться на наличие ProtoMessage для операций с указанными структурами. Например, метод proto.Marshal может использовать это для валидации перед сериализацией. 
func (*HelloRequest) ProtoMessage() {}

// Зачем нужен ProtoReflect?
// 1. **Интроспекция сообщения**:
//    - Позволяет получить доступ к метаданным сообщениям.
//    - Позволяет динамически анализировать и изменять сообщения без необходимости ручного написания кода для каждого поля.
// 2. **Универсальные операции**:
//    - Используется внутри различных библиотек и инструментов, которые работают с Protobuf сообщениями для выполнения сериализации, десериализации, сравнения, создания копий и других операций.
// 3. **Типовой интерфейс**:
//    - Предоставляет унифицированный способ взаимодействия с различными сообщениями Protobuf. Это особенно полезно, если вы работаете с множеством различных типов сообщений.
func (x *HelloRequest) ProtoReflect() protoreflect.Message {
    mi := &file_proto_service_proto_msgTypes[0]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Метод Descriptor — это часть сгенерированного кода Protobuf, который предоставляет доступ к метаданным сообщениям.
// Descriptor — это описание структуры Protobuf сообщения. Он содержит:
// - Имя сообщения
// - Имя каждого поля
// - Тип каждого поля
// - Порядковые номера полей
// - Другую метаинформацию
// Метод Descriptor предоставляет доступ к этой информации в сжатом виде, и используется внутри различных библиотек, которые работают с Protobuf сообщениями для анализа и манипуляции ими.
// Зачем нужен метод Descriptor?
// 1. **Доступ к метаданным**: 
//    - Метод предоставляет доступ к метаданным сообщения в сжатом виде, которые могут быть использованы для дальнейшего анализа или манипуляции.
// 2. **Рефлексия**: 
//    - Позволяет рефлективно получить информацию о структуре сообщения. Это позволяет динамически анализировать сообщения без необходимости явно знать их структуру заранее.
// 3. **Сжатие метаданных**: 
//    - Использование метода file_proto_service_proto_rawDescGZIP() обеспечивает доступ к метаданным в сжатом формате GZIP, что экономит память и время передачи данных.
// Пример использования:
// func printFieldDescriptors(msg proto.Message) {
//    descriptor := msg.ProtoReflect().Descriptor()
//    fields := descriptor.Fields()
//    for i := 0; i < fields.Len(); i++ {
//        field := fields.Get(i)
//        fmt.Printf("Field Name: %s, Field Type: %s\n", field.Name(), field.Kind())
//    }
// }
// 
// func main() {
//    req := &pb.HelloRequest{Name: "John"}
//    printFieldDescriptors(req)
}
func (*HelloRequest) Descriptor() ([]byte, []int) {
// возвращает сжатые данные, которые могут быть распакованы для получения полного дескриптора сообщения.
// []int{0} указывает на индекс HelloRequest в массиве сообщений. Это позволяет быстро найти дескриптор для конкретного сообщения в массиве.
    return file_proto_service_proto_rawDescGZIP(), []int{0}
}

func (x *HelloRequest) GetName() string {
    if x != nil {
        return x.Name
    }
    return ""
}
```

   - Вспомогательные функции для работы с сообщениями

   Этот файл генерируется плагином `protoc-gen-go`.

`proto/service_grpc.pb.go`

  Этот файл содержит Go код, специфичный для gRPC. Он включает:
  
   - Определения интерфейсов клиента и сервера для вашего gRPC сервиса

```go
type GreeterClient interface {
        SayHello(ctx context.Context, in *HelloRequest, opts ...grpc.CallOption) (*HelloReply, error)
}

type GreeterServer interface {
        SayHello(context.Context, *HelloRequest) (*HelloReply, error)
        mustEmbedUnimplementedGreeterServer()
}

```

   - Клиентские заглушки (stubs)

```go
   type GreeterClient interface {
       SayHello(ctx context.Context, in *HelloRequest, opts ...grpc.CallOption) (*HelloReply, error)
   }

   type greeterClient struct {
       cc grpc.ClientConnInterface
   }
   
   func (c *greeterClient) SayHello(ctx context.Context, in *HelloRequest, opts ...grpc.CallOption) (*HelloReply, error) {
       cOpts := append([]grpc.CallOption{grpc.StaticMethod()}, opts...)
       out := new(HelloReply)
       err := c.cc.Invoke(ctx, Greeter_SayHello_FullMethodName, in, out, cOpts...)
       if err != nil {
           return nil, err
       }
       return out, nil
   }
   
```

   - Регистрационные функции для серверной части

```go
func RegisterGreeterServer(s grpc.ServiceRegistrar, srv GreeterServer) {
        s.RegisterService(&Greeter_ServiceDesc, srv)
}
```

- И еще много обвязок чтобы все работало

 Этот файл генерируется плагином `protoc-gen-go-grpc`.

Эти файлы автоматически создаются на основе .proto файла и содержат всю необходимую логику для работы с Protocol Buffers и gRPC в Go. 

Использование этих файлов:

- В серверном коде будем реализовывать интерфейсы, определенные в `service_grpc.pb.go`.
- В клиентском коде будем использовать клиентские заглушки из `service_grpc.pb.go` для вызова удаленных процедур.
- Структуры сообщений из `service.pb.go` будут использоваться как для сериализации/десериализации данных, так и для передачи аргументов в gRPC вызовах.

`server/main.go`

```go

package main

import (
        "context"
        "log"
        "net"
        pb "myservice/proto" // импортируем сгенерированные protobuf код и алиасим его как pb
        "google.golang.org/grpc"
)

// Определяем структуру server, которая будет реализовывать интерфейс сгенерированного сервера gRPC. Встраиваем pb.UnimplementedGreeterServer для совместимости и для того, чтобы нам не нужно было реализовывать все методы интерфейса.
type server struct {
        pb.UnimplementedGreeterServer 
}

func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
        return &pb.HelloReply{Message: "Hello " + in.Name}, nil
}

func main() {
        lis, err := net.Listen("tcp", ":50052")
        if err != nil {
                log.Fatalf("failed to listen: %v", err)
        }
        s := grpc.NewServer()
        pb.RegisterGreeterServer(s, &server{})
        log.Printf("server listening at %v", lis.Addr())
        if err := s.Serve(lis); err != nil {
                log.Fatalf("failed to serve: %v", err)
        }
}

```

`client/main.go`
```go

package main

import (
        "context"
        "log"
        "time"

        pb "myservice/proto"

        "google.golang.org/grpc"
)

func main() {
        conn, err := grpc.Dial("localhost:50052", grpc.WithInsecure(), grpc.WithBlock())
        if err != nil {
                log.Fatalf("did not connect: %v", err)
        }
        defer conn.Close()
        c := pb.NewGreeterClient(conn)

        ctx, cancel := context.WithTimeout(context.Background(), time.Second)
        defer cancel()
        r, err := c.SayHello(ctx, &pb.HelloRequest{Name: "World"})
        if err != nil {
                log.Fatalf("could not greet: %v", err)
        }
        log.Printf("Greeting: %s", r.GetMessage())
}
```

- Запускаем!
```bash
go mod init myservice
go mod tidy

go run server/main.go
2024/07/04 12:23:27 server listening at [::]:50052

go run client/main.go
2024/07/05 00:33:44 Greeting: Hello World
```

- На этом все, но все же есть еще пара слов :)
- Как вы знаете, есть прекрасная утилита `curl`, которая позволяет делать http запросы.
Для grpc тоже есть подобная утилита https://github.com/fullstorydev/grpcurl

```bash
$grpcurl -plaintext -import-path ./proto -proto service.proto localhost:50052 list

myservice.Greeter

$grpcurl -plaintext -import-path ./proto -proto service.proto localhost:50052 list myservice.Greeter

myservice.Greeter.SayHello

$grpcurl -plaintext -import-path ./proto -proto service.proto localhost:50052 describe myservice.Greeter.SayHello

myservice.Greeter.SayHello is a method:
rpc SayHello ( .myservice.HelloRequest ) returns ( .myservice.HelloReply );

$grpcurl -plaintext -import-path ./proto -proto service.proto localhost:50052 describe myservice.HelloRequest

myservice.HelloRequest is a message:
message HelloRequest {
  string name = 1;
}

$grpcurl -d '{"name": "azalio"}' -plaintext -import-path ./proto -proto service.proto localhost:50052 myservice.Greeter.SayHello

{
  "message": "Hello azalio"
}
```
- Как можно заметить, мне пришлось указать контракт `./proto` иначе я не получал данные от сервера. Чтобы этого не делать, в код можно включить gRPC reflection (рефлексию), которая может динамически отображать список сервисов.
