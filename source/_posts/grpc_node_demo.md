---
title: gRPC node demo
date: 2018/07/08 15:00:00
tags:
  - gRPC
categories: gRPC
---

在看完gRPC定义之后，再根据官方demo继续深入了解一下gRPC

## 初始化
```plain
git clone https://github.com/grpc/grpc

cd grpc/examples/node/dynamic_codegen

npm install
```

Note：官方是使用的v1.17.1的版本
另外，我使用的是Node demo，需要提前安装好Node

<!-- more -->

## 运行gRPC
### Client
```plain
  if (process.argv.length >= 3) {
    user = process.argv[2];
  } else {
    user = 'world';
  }
  client.sayHello({name: user}, function(err, response) {
    console.log('Greeting:', response.message);
  });
```

如果没有多余参数，user默认为world，client会将{name: user}发送至服务端

### Server
```plain
callback(null, { message: "Hello " + call.request.name });
```

gRPC接受到请求后，重新包装数据

### 运行

```plain
node greeter_server.js

node greeter_client.js
```

分别运行gRPC的服务端及客户端，下面是客户端的输出：



![image.png | left | 754x83](https://cdn.nlark.com/yuque/0/2018/png/92822/1545543713785-9857b73e-1a58-4dd6-89b8-cd23afb6dba0.png "")


## 升级gRPC服务
### 原有 .proto 定义

```plain
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

### 升级 .proto
添加sayHelloAgain方法：

```plain
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
  // Sends another greeting
  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

### Server 中实现新的方法

```plain
function sayHelloAgain(call, callback) {
  callback(null, {message: 'Hello again, ' + call.request.name});
}

function main() {
  var server = new grpc.Server();
  server.addProtoService(hello_proto.Greeter.service,
                         {sayHello: sayHello, sayHelloAgain: sayHelloAgain});
  server.bind('0.0.0.0:50051', grpc.ServerCredentials.createInsecure());
  server.start();
}
```

### Client 中调用新的方法

```plain
  client.sayHelloAgain({ name: "you" }, function(err, response) {
    console.log("Greeting:", response.message);
  });
```

### 运行
运行方式还与之前相同：

```plain
node greeter_server.js

node greeter_client.js
```

新的返回：



![image.png | left | 683x64](https://cdn.nlark.com/yuque/0/2018/png/92822/1545550682867-59ff4627-b1f2-4c4c-8bf0-6262df4b7315.png "")


## 资料
* [https://grpc.io/docs/quickstart/node.html#download-the-example](https://grpc.io/docs/quickstart/node.html#download-the-example) 
* demo地址：[https://github.com/JianmingXia/StudyTest/tree/master/gRPC](https://github.com/JianmingXia/StudyTest/tree/master/gRPC)
