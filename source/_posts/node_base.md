---
title: 最简单的 NodeJS 项目
date: 2018/06/30 18:30:00
tags:
  - NodeJS
categories: NodeJS
---

## 创建一个服务
### createServer
在 Node 中，通常使用 createServer 来创建服务：

```
const http = require('http');

const server = http.createServer((request, response) => {
  // magic happens here!
});
```

每当服务接收到 HTTP 请求时，createServer 中的 Function 就被自动执行——这个 Function 也被称为是请求处理函数。<br />值得注意的是，createServer 返回的是一个 EventEmitter，默认会被添加 request event：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550978156056-79b8f1e3-0ea9-4c2f-8df2-27a0ea20e634.png#align=left&display=inline&height=367&linkTarget=_blank&name=image.png&originHeight=734&originWidth=2692&size=182977&status=done&width=1346)

故而上面的代码可以改为：

```
const server = http.createServer();
server.on('request', (request, response) => {
  // the same kind of magic happens here!
});
```

### listen
创建服务器之后，还需要为服务器设置监听的端口：

```
server.listen(8889);
```

更多操作见[文档](https://nodejs.org/api/net.html#net_server_listen)。

## 请求
### 方法 & 访问地址 & 请求头
requestListener 即为 request event 的回调函数，它的类型为 RequestListener：

```
type RequestListener = (req: IncomingMessage, res: ServerResponse) => void;
```

而当处理请求时，首先要看请求方法及url，以此服务器来决定做何处理。<br />Node 将这些属性放在了 request 对象之中，可以直接从 request 对象中取出来：

```
const { method, url } = request;

// 获取请求头
const { headers } = request;
const userAgent = headers['user-agent'];
```

**更多属性**：

```
// request 类型
class IncomingMessage extends stream.Readable {
    constructor(socket: net.Socket);

    httpVersion: string;
    httpVersionMajor: number;
    httpVersionMinor: number;
    connection: net.Socket;
    headers: IncomingHttpHeaders;
    rawHeaders: string[];
    trailers: { [key: string]: string | undefined };
    rawTrailers: string[];
    setTimeout(msecs: number, callback: () => void): this;
    /**
    * Only valid for request obtained from http.Server.
    */
    method?: string;
    /**
    * Only valid for request obtained from http.Server.
    */
    url?: string;
    /**
    * Only valid for response obtained from http.ClientRequest.
    */
    statusCode?: number;
    /**
    * Only valid for response obtained from http.ClientRequest.
    */
    statusMessage?: string;
    socket: net.Socket;
    destroy(error?: Error): void;
}

// 请求头
// incoming headers will never contain number
interface IncomingHttpHeaders {
    'accept'?: string;
    'accept-patch'?: string;
    'accept-ranges'?: string;
    'access-control-allow-credentials'?: string;
    'access-control-allow-headers'?: string;
    'access-control-allow-methods'?: string;
    'access-control-allow-origin'?: string;
    'access-control-expose-headers'?: string;
    'access-control-max-age'?: string;
    'age'?: string;
    'allow'?: string;
    'alt-svc'?: string;
    'authorization'?: string;
    'cache-control'?: string;
    'connection'?: string;
    'content-disposition'?: string;
    'content-encoding'?: string;
    'content-language'?: string;
    'content-length'?: string;
    'content-location'?: string;
    'content-range'?: string;
    'content-type'?: string;
    'cookie'?: string;
    'date'?: string;
    'expect'?: string;
    'expires'?: string;
    'forwarded'?: string;
    'from'?: string;
    'host'?: string;
    'if-match'?: string;
    'if-modified-since'?: string;
    'if-none-match'?: string;
    'if-unmodified-since'?: string;
    'last-modified'?: string;
    'location'?: string;
    'pragma'?: string;
    'proxy-authenticate'?: string;
    'proxy-authorization'?: string;
    'public-key-pins'?: string;
    'range'?: string;
    'referer'?: string;
    'retry-after'?: string;
    'set-cookie'?: string[];
    'strict-transport-security'?: string;
    'tk'?: string;
    'trailer'?: string;
    'transfer-encoding'?: string;
    'upgrade'?: string;
    'user-agent'?: string;
    'vary'?: string;
    'via'?: string;
    'warning'?: string;
    'www-authenticate'?: string;
    [header: string]: string | string[] | undefined;
}
```

有一点值得注意，**所有的请求头都是小写**，也方便去了解析请求头。

### 请求体
当接收到 POST/PUT/PATCH 等请求时，请求体对服务非常重要。但是与获取请求头相比，获取请求体会略微麻烦一点。但在上面的代码中可以发现，request 继承自 stream.Readable，所以可以通过添加 data、end 事件将数据取出来。

每次 data 事件触发时获取的数据块皆为一个 Buffer；如果已知数据为字符串，可以在 data 事件中先将数据收集起来，然后在 end 事件中进行拼接：

```
let body = [];
request.on('data', (chunk) => {
  body.push(chunk);
}).on('end', () => {
  body = Buffer.concat(body).toString();
  // at this point, `body` has the entire request body stored in it as a string
});
```

## 响应
如果只处理了请求，而没响应，如果在浏览器上运行，最终获得的就是一个超时

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550988062446-9716363a-2960-4fa4-a524-d5c79f79d6ed.png#align=left&display=inline&height=156&linkTarget=_blank&name=image.png&originHeight=312&originWidth=2292&size=96223&status=done&width=1146)

### response

```
// https://github.com/nodejs/node/blob/master/lib/_http_server.js#L108-L256
class ServerResponse extends OutgoingMessage {
    statusCode: number;
    statusMessage: string;

    constructor(req: IncomingMessage);

    assignSocket(socket: net.Socket): void;
    detachSocket(socket: net.Socket): void;
    // https://github.com/nodejs/node/blob/master/test/parallel/test-http-write-callbacks.js#L53
    // no args in writeContinue callback
    writeContinue(callback?: () => void): void;
    writeHead(statusCode: number, reasonPhrase?: string, headers?: OutgoingHttpHeaders): void;
    writeHead(statusCode: number, headers?: OutgoingHttpHeaders): void;
}

// https://github.com/nodejs/node/blob/master/lib/_http_outgoing.js
class OutgoingMessage extends stream.Writable {
    upgrading: boolean;
    chunkedEncoding: boolean;
    shouldKeepAlive: boolean;
    useChunkedEncodingByDefault: boolean;
    sendDate: boolean;
    finished: boolean;
    headersSent: boolean;
    connection: net.Socket;

    constructor();

    setTimeout(msecs: number, callback?: () => void): this;
    setHeader(name: string, value: number | string | string[]): void;
    getHeader(name: string): number | string | string[] | undefined;
    getHeaders(): OutgoingHttpHeaders;
    getHeaderNames(): string[];
    hasHeader(name: string): boolean;
    removeHeader(name: string): void;
    addTrailers(headers: OutgoingHttpHeaders | Array<[string, string]>): void;
    flushHeaders(): void;
}
```

### 状态码
可以直接通过设置 statusCode 设置返回码

```
response.statusCode = 404; // Tell the client that the resource wasn't found.
```

### 响应头

```
response.setHeader('Content-Type', 'application/json');
response.setHeader('X-Powered-By', 'bacon');
```

设置响应头时，它们的名字是大小写敏感的。如果重复设置了响应头，最后一次设置的值即为系统得到的值。

### writeHead
writeHead 可以同时设置 状态码 & 响应头：

```
response.writeHead(200, {
  'Content-Type': 'application/json',
  'X-Powered-By': 'bacon'
});
```

### 响应体
既然 response 是一个 WriteableStream，写响应体就简单了：

```
response.write('<html>');
response.write('<body>');
response.write('<h1>Hello, World!</h1>');
response.write('</body>');
response.write('</html>');
response.end();
```

另外，end 方法也是可以带数据，故而上面的代码可以简化为：

```
response.end('<html><body><h1>Hello, World!</h1></body></html>');
```

## 代码
### 示例代码

```
var http = require("http");

http.createServer((request, response) => {
  const { headers, method, url } = request;
  let body = [];
  request.on('error', (err) => {
    console.error(err);
  }).on('data', (chunk) => {
    body.push(chunk);
  }).on('end', () => {
    body = Buffer.concat(body).toString();

    response.on('error', (err) => {
      console.error(err);
    });

    response.writeHead(200, {'Content-Type': 'application/json'})

    const responseBody = { headers, method, url, body };

    response.end(JSON.stringify(responseBody))
  });
}).listen(8080);

console.log("Server running at http://127.0.0.1:8080/");
```

运行情况：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550989262112-9b5af151-9269-46ff-862a-a147b0927987.png#align=left&display=inline&height=228&linkTarget=_blank&name=image.png&originHeight=456&originWidth=2050&size=138892&status=done&width=1025)

### 简化代码
只对以下请求做处理：
* POST 请求
* url 为 /echo

```
const http = require('http');

http.createServer((request, response) => {
  if (request.method === 'POST' && request.url === '/echo') {
    let body = [];
    request.on('data', (chunk) => {
      body.push(chunk);
    }).on('end', () => {
      body = Buffer.concat(body).toString();
      response.end(body);
    });
  } else {
    response.statusCode = 404;
    response.end();
  }
}).listen(8080);
```

#### 失败请求

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550990316267-52849b14-5e5d-421a-b534-c555f8a3faac.png#align=left&display=inline&height=102&linkTarget=_blank&name=image.png&originHeight=204&originWidth=1162&size=41501&status=done&width=581)

#### 成功请求

![image.png](https://cdn.nlark.com/yuque/0/2019/png/92822/1550990337884-bd963d38-2890-46b8-80f1-196e82a42518.png#align=left&display=inline&height=122&linkTarget=_blank&name=image.png&originHeight=244&originWidth=1262&size=50702&status=done&width=631)

### 继续优化
request 是一个可读流，而 response 是一个可写流——可以通过 pipe 直接流转：

```
const http = require('http');

http.createServer((request, response) => {
  if (request.method === 'POST' && request.url === '/echo') {
    request.pipe(response);
  } else {
    response.statusCode = 404;
    response.end();
  }
}).listen(8080);
```

## 补充
### request
request 不只是一个 ReadableStream 对象，同时也是一个 EventEmitter。当有错误在 request 对象在发生时，会触发自身的 error 事件，如果事件未被捕捉，错误会被抛出，进而导致崩溃

```
request.on('error', (err) => {
  // This prints the error message and stack trace to `stderr`.
  console.error(err.stack);
});
```

### response
与 request 类似，response 上同样需要进行处理

## 资料
* [https://nodejs.org/zh-cn/docs/guides/anatomy-of-an-http-transaction/](https://nodejs.org/zh-cn/docs/guides/anatomy-of-an-http-transaction/)
