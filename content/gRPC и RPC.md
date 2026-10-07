# gRPC и RPC

**RPC** (Remote Procedure Calls) отличается от REST тем, что прячет под собой описание контрактов и вызовы удалённых методов так, что кажется, будто мы вызываем методы в своей программе.

**gRPC** — подвид RPC от Google, его формат передачи данных называется protocol buffers (protobuf) — https://protobuf.dev.

С помощью тулинга контракты описываются один раз через proto definition, затем генерируются необходимые классы под языки программирования и раздаются клиенту и серверу.

Формат protobuf бинарный — это даёт ему преимущество в скорости сериализации/десериализации, а также в скорости передачи данных по сети. Из минусов — нельзя глазами посмотреть, что лежало в payload.

## Пример описания .proto файла

Документация: https://protobuf.dev/programming-guides/proto3/

```proto
message HelloRequest {
  string name = 1;
  string description = 2;
  int32 id = 3;
}

message HelloResponse {
  string processedMessage = 1;
}

service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}
```

## Формы общения клиента и сервера

gRPC поддерживает следующие формы:

- **Unary** (обычный вызов процедур)
- **Client side streaming**
- **Server side streaming**
- **Bidirectional streaming**

Так как общение идёт через протокол HTTP/2, не требуется каждый раз устанавливать новое TCP-соединение с трёхсторонним рукопожатием. Все запросы могут использовать одно и то же соединение.

Стриминг в HTTP 1.1 не закрывает TCP-соединение и не мониторит его.

---

Статья в блоге gRPC про .NET Core: https://grpc.io/blog/grpc-on-dotnetcore/
