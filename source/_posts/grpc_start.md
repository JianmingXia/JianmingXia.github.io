---
title: gRPC 基础
date: 2018/07/01 12:00:00
tags:
  - gRPC
categories: gRPC
---

## 说在前面
首先为自己的目光短浅感到羞愧，一直待在自己的舒适区过得很舒服，没有接触什么新的技术或是拓宽自己的视野。在有次出去面试介绍参与的项目中：描述服务之间通过Socket连接传输数据，而数据则是通过服务共同依赖的Server-Lib服务中定义的pack 及 unpack函数进行传输（pack及unpack当作做write 及 read操作即可）。举个例子：

```plain
	// 写数据
    reply_data->writeInt32(_company_id);
	reply_data->writeString(_company_name);
	reply_data->writeInt32(_creator_passport_id);
	reply_data->writeString(_icon_url);
	reply_data->writeInt32(_edition_type);

    // 读数据
	_company_id = data->readInt32();
	_company_name = data->readString();
	_creator_passport_id = data->readInt32();
	_icon_url = data->readString();
	_edition_type = data->readInt32();
```
<!-- more -->

当时的面试官get 到之后，问我为什么你们不使用Protocol Buffer 呢？当时的黑人问号脸真的是太尴尬了。。。
当然回去之后我就认真补了课，在接下来的工作也应用了，下面来聊一聊。

## gRPC 简单介绍
gRPC可以使用protocol buffers作为其接口定义语言(IDL)和底层消息交换格式。

在gRPC中，客户端应用程序可以直接调用不同机器上服务器应用程序上的方法，就像它是一个本地对象一样，这使得我们更容易创建分布式应用程序和服务。在许多RPC系统中，gRPC基于定义服务的思想，指定可以通过参数和返回类型远程调用的方法：
* 在服务端，服务实现这个接口，并运行gRPC服务器来处理客户端调用
* 在客户端，客户端有一个存根（在某些语言中仅称为客户端）提供与服务端相同的方法



![image.png | left | 552x327](https://cdn.nlark.com/yuque/0/2018/png/92822/1545487408776-3ea32a85-2f87-4868-b9d7-7c022af2ec69.png "")



gRPC客户端和服务端可以在各种环境中运行并相互通信——从谷歌内部服务到自己的桌面——并且可以用gRPC支持的任何语言编写。例如，你可以轻松地用Java创建gRPC服务端，客户端使用Go、Python或Ruby。

## Protocol Buffers
> [Protocol Buffer文档](https://developers.google.com/protocol-buffers/docs/overview)
默认情况下，gRPC使用protocol buffers，这是谷歌用于序列化结构化数据的成熟开源机制（尽管它可以与JSON等其他数据格式一起使用）。

使用协议缓冲区时的第一步是要在proto文件中定义想要序列化数据的结构：这是一个扩展名为.proto的普通文本。Protocol Buffer数据以 message 构造，其中每个message都是包含一系列名称-值对信息的小逻辑记录。如：

```plain
message Person {
  string name = 1;
  int32 id = 2;
  bool has_ponycopter = 3;
}
```

然后，一旦指定了数据结构，就可以使用protocol buffer编译器 __protoc__ 从proto定义生成首选语言中的数据访问类。这些方法为每个字段（如name()和set\_name()）提供简单的访问器，以及将整个结构序列化/解析为/从原始字节——例如，如果选择的语言是c++，那么在上面的示例上运行编译器将生成一个名为Person的类。

## gRPC 概念
### 服务定义
与许多RPC系统一样，gRPC基于定义服务的思想，指定可以通过参数和返回类型远程调用的方法：

```plain
service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}

message HelloRequest {
  string greeting = 1;
}

message HelloResponse {
  string reply = 1;
}
```

gRPC可以定义四种服务方式：
* Unary RPCs：客户端向服务端发送一个请求，并获得一个响应，就像普通的函数调用一样
    ```plain
    rpc SayHello(HelloRequest) returns (HelloResponse){
    }
    ```
* Server streaming RPCs：其中客户端向服务端发送请求并获取流以读取返回的消息序列。客户端从返回的流中读取信息，直到没有更多的消息为止。gRPC保证了单个RPC调用中的消息排序
    ```plain
    rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse){
    }
    ```
* Client streaming RPCs：客户端在其中写入一系列消息并将其发送到服务器，同样使用提供的流。一旦客户端完成了消息的编写，它将等待服务端读取它们并返回其响应。同样，gRPC保证在单个RPC调用中对消息进行排序
    ```plain
    rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse) {
    }
    ```
* Bidirectional streaming RPCs：双方使用读写流发送消息序列。这两个流独立操作，因此客户端和服务端可以按照自己喜欢的顺序读写。例如，服务端可以在写入响应之前等待接收所有客户端消息，或者它可以交替读取消息然后写入消息，或者其他一些读和写的组合。保留每个流中的消息顺序
    ```plain
    rpc BidiHello(stream HelloRequest) returns (stream HelloResponse){
    }
    ```

### API surface 应用
从 .proto 文件中的服务定义开始，gRPC提供了生成客户端和服务器端代码的protocol buffer编译器插件。gRPC使用者通常在客户端调用这些API，并在服务器端实现相应的API：
* 在服务器端，服务器实现服务声明的方法，并运行gRPC服务端来处理客户端调用。gRPC基础设施解码传入的请求，执行服务方法，并对服务响应进行编码
* 在客户端，客户端有一个称为存根的本地对象(对于某些语言，首选术语是客户端)，它实现了与服务相同的方法。然后客户端可以在本地对象上调用这些方法，将调用的参数包装在适当的protocol buffer message类型中——gRPC在将请求发送到服务端并对返回的服务端protocol buffer响应进行处理

### 同步与异步
在服务器响应到达之前，阻塞的同步RPC调用最接近于RPC渴望的过程调用的抽象。另一方面，网络本质上是异步的，在许多情况下，能够在不阻塞当前线程的情况下启动rpc非常有用。

大多数语言中的gRPC编程都有同步和异步两种风格。

## 小结
* 像调用本地服务一样调用远程服务
* 适用于低延迟、高扩展性、分布式的系统
* 支持多种语言

## 资料
* [https://grpc.io/](https://grpc.io/)
* [https://grpc.io/docs/guides/index.html](https://grpc.io/docs/guides/index.html)
* [https://grpc.io/docs/guides/concepts.html](https://grpc.io/docs/guides/concepts.html)
* [https://developers.google.com/protocol-buffers/docs/overview](https://developers.google.com/protocol-buffers/docs/overview)

