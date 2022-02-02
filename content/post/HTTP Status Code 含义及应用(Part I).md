---
title: "HTTP Status Code 含义及应用(Part I)"
date: 2022-02-01T22:50:18+08:00
author: walker
---

翻译自[原文](https://datatracker.ietf.org/doc/html/rfc7231#section-6)

## 1XX

1XX的HTTP Status Code只作为临时响应，并且，HTTP 1.0 的Client并不支持这个1XX的状态码，对于代理而言，代理需要转发1XX的HTTP Status Code

### 100 Continue

Server通知Client请求已经已经被接收且Client 需要继续将未发送完毕的请求发送到Server端，如果Client已经发送完所有的请求，忽略此响应。

100 这个Status Code可以用于作为Pre-Check HTTP Request， 例如说Client准备发送一个很大请求体的请求，Client可以通过Header检查请求是否合法来进行一次p re check，如果check通过，Client再进行接下来的操作。

### 101 Switch Protocals

Server 收到了ClientSwich Protocals的请求并且准备根据Client的Header切换协议，例如[Web Socket](https://datatracker.ietf.org/doc/html/rfc6455#section-1.2)的握手请求Header和Response的Header : 

```pre
   The handshake from the client looks as follows:

        GET /chat HTTP/1.1
        Host: server.example.com
        Upgrade: websocket
        Connection: Upgrade
        Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
        Origin: http://example.com
        Sec-WebSocket-Protocol: chat, superchat
        Sec-WebSocket-Version: 13

   The handshake from the server looks as follows:

        HTTP/1.1 101 Switching Protocols
        Upgrade: websocket
        Connection: Upgrade
        Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
        Sec-WebSocket-Protocol: chat

```

## 2XX

此类的Status Code表明Server 收到，接受并且成功处理了Client的请求。

### 200 OK

请求被接受。对于处理结果，对于不同的请求类型的含义不同

1. GET :  Client需要的Resource被响应，Response Body中会含有Client需要的Resource资源
2. POST : Client发送给Server的请求被正确处理, Response Body中可以含有对Client Post资源的处理描述
3. PUT：Client 发送给Server的Body被成功接收
4. DELETE : Client想要删除掉的资源已被成功删除

### 201 Ceated 

表示Client想要在Server端创建的资源已被正确创建，正由于此，201一般会被用来作为对POST/PUT请求的响应。相对于200低于POST / PUT的处理在于，Response必须要包含有一条信息对创建资源具体位置的描述(URI)。

**一般是以一个`ETag`响应头来描述被创建资源的位置。**

### 202 Accepted

202 Status Code用于表明Client的请求已经被接收，Server可能正在进行处理没有完成。**一般用于异步处理的操作**，但是由于HTTP Request的机制，Server无法通知到Client具体的处理结果，因为异步操作可能遇到失败的情况。

### 203 Non-Authoritative Information

表示Server正确处理了Client的请求，但是响应的信息是从一系列第三方获取到的。这个Status Code一般用于Proxy-Server对于Client的请求处理

### 204 No Content

204 Status Code的表示的含义表明其Response Body没有多余的信息响应给Client，响应中的Metadata信息可能会指代Request的请求的完成后的指示。例如，一个PUT请求操作将一个文件上传后，会有一个ETag Header指向创建后的文件URI，但是相比于201 Created, 其Response Body是空的。

对于Web页面一般用于用户的`SAVE` 操作

### 205 Reset Content

Server 收到了Client的请求并成功执行了它，并且响应了一个空的Response Body, 但是不同于204 响应，此响应要求Client  Reset Document View。



## 3xx

3XX 重定向的Status Code 表明Client需要采取进一步的动作才能可以完成请求操作。

重定向类型：

1. 重定向表明Client请求的资源在其他地方是available的，正如响应的Header中Location值表明的那样。301(Move Permanently)，302(Found) 和 307(Temporary Redirect)。 都是属于这个类型
2. 重定向响应给了Client多个选项，300(Multiple Choice)属于这个类型。
3. 重定向到由 Location 标识的不同资源 字段，可以表示对请求的间接响应，如 在 303（See Other）状态代码中。
4. 重定向到以前缓存的结果，如 304（Not Modified）状态码。

### 300 Mutiple Choice

Server 需要一个Payload 来提供一个list让Client选择一个更合适的重定向链接地址。*使用场景有限*

### 301 Move Permanently

Client请求的资源被永久转移到Server响应的Location Header指定的地址，Client通过自动跳转到指定的URI地址。

另外 ： 

1. 服务器的响应负载通常包含一个简短的 带有指向新 URI 的超链接的超文本注释。
2. 由于历史原因，用户代理可能会更改请求 从 POST 到 GET 的方法用于后续请求。如果这个不是期望行为，307（Temporary Redirect）状态代码 可以代替使用。

### 302 Found

302（Found）状态码表示目标资源 临时驻留在不同的 URI 下。由于重定向有时可能会更改，客户应继续使用 未来请求的有效请求 URI。




