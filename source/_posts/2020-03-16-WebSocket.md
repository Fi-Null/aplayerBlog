---
title: WebSocket
date: 2020-03-16 14:34:29
category:
- webSocket
---

## 什么是WebSocket呢？

WebSocket是HTML5新增的一种通信协议，目标主流的浏览器都支持这个协议，比如Google的Chrome、Apple的Safari、Mozala的Firefox、Microsoft的IE等。对WebSocket协议支持最早的当属Chrome浏览器，从Chrome12开始就已经开始支持，随着协议草案不断完善，各个浏览器对协议的实现也在不停的更新。

## 为什么会引入WebSocket协议呢？

浏览器已经支持HTTP协议了，为什么还要开发一种新的WebSocket协议呢？因为HTTP协议是一种单向的网络协议，在建立连接后只允许浏览器或用户代理(`UserAgent`)向Web服务器发出请求资源后，Web服务器才能返回相应的资源数据。而Web服务器是不能够主动推送数据给浏览器的，HTTP设计之初的考虑到安全问题，如果Web服务器能够主动的推送数据给浏览器，那么浏览器就太容易受到攻击，一些广告商也会主动的将广告信息在不经意间强行推送给用户，这不能不说是一个灾难。但是单向的HTTP协议给现代的网站和Web应用程序却带来了许多问题。加入要开发一个基于Web的应用程序去获取当前Web服务器的实时数据的话，比如股票的实时行情，火车票的剩余票数等，这个时候就需要浏览器与Web服务器之间反复的进行HTTP通信，浏览器需要不断地发送请求去获取实时数据。

## 实时获取Web服务器资源的方式有哪几种呢？

那么在还没有WebSocket协议之前，有哪几种方式可以实时的获取Web服务器上的资源数据呢？

1. 短轮询`Polling`

Polling的方式是通过浏览器定时向Web服务器发送HTTP请求，Web服务器接收到请求后会将最新的数据返还给浏览器。浏览器得到数据后将其渲染显示，然后再定期的重复这一过程。

![](https://raw.githubusercontent.com/Fi-Null/blog-pic/master/blog-pics/netty/websocket.png)

Polling的方式虽然能够满足实时的需求，但存在一定的问题，比如在某段时间内Web服务器没有数据更新呢，此时浏览器仍然需要定时发送请求过来询问，Web服务器会将以前的老数据再次传送过去，浏览器将这些没有变化的数据又渲染显示出来。**这样既浪费了网络带宽，又浪费了CPU的利用利率。**如果将浏览器发送请求的时间周期调大一些，虽然可以缓解这一问题，但如果在Web服务器上数据更新很快时，又将无法保证Web应用程序获取数据的实时性。

针对这种情况，Polling做出改进而衍生出来Long Polling。

2. 长轮询`Long Polling` 

Long Polling的操作是这样的：浏览器发送请求到Web服务器时，Web服务器可以做两件事情。第一件事是如果服务器数据更新就会立即将数据发回给浏览器，浏览器接收到数据后再理解发送请求给Web服务器。第二件事是如果服务器没有数据更新，此时与Polling不同的是Web服务器不会立即发送回应信息给浏览器，而会见这个请求保持住，等到有数据更新时，再来响应这个请求。当然，如果服务器的数据长期没有更新的话，一段时间后，这些请求就会超时，浏览器将会收到超时消息，当浏览器收到超时消息又会立即发送一个新的请求给Web服务器，然后依次循环这个过程。

![](https://raw.githubusercontent.com/Fi-Null/blog-pic/master/blog-pics/netty/websocket1.png)

Long Polling的方式虽然在某种程度上减小了网络带宽和CPU的利用率等问题，但仍存在缺陷，比如Web服务器的数据更新速度较快，当Web服务器在传送一个数据包给浏览器后，必须等待浏览器的下一个请求的到来才能传递第二给更新的数据包给浏览器。这样的话，浏览器显示的实时数据最快的时间也就是2 x RTT(往返时间)。另外，由于HTTP数据包的头部数据量往往会很大，一般有400多字节，但是真正被服务器使用的却很少，有时只有10字节左右，这样的数据包在网络上周期性的传输，难免对网络带宽又是一种浪费。

实际上Long Polling长轮询的底层实现是在服务器的程序中加入一个死循环，在循环中检测数据的变化，当发现有数据时会立即将其输出给浏览器并断开连接，浏览器收到数据后会再次发起请求进入下一个周期。

**长轮询的弊端是服务器长时间连接会消耗服务器资源，另外返回的数据的顺序无法保证，难以管理和维护。**

对于长轮询的处理，服务器并不会一直保持，通常的做法是会设置一个最大时限，可以通过心跳包的方式，设置多少秒之后没有接收到心跳包就关闭当前连接。

### 总结

> http协议的特点是服务器不能主动联系客户端，只能由客户端发起。它的被动性预示了在完成双向通信时需要不停的连接或连接一直打开，这就需要服务器快速的处理速度或高并发的能力，是非常消耗资源的。

通过以上的分析可知，要想在浏览器上支持双向通信而且协议的头部又不是那么的庞大，不得不采用新的协议，WebSocket也就是为了解决这个问题而设计诞生的。

## WebSocket协议

WebSocket协议是一种双向的通信协议，它建立在TCP之上，同HTTP一样是通过TCP来传递数据的，不过它与HTTP最大的不同点在于：

1. WebSocket是一种双向通信协议，在建立连接后，WebSocket服务器和浏览器之间都能主动地向对象发送或接收数据，这就像Socket一样，只是与之不同的是，WebSocket是一种建立在Web基础上的简单模拟Socket的协议。

2. WebSocket需要通过握手建立连接，类似于TCP也需要客户端和服务端进行握手成功后才能互相通信。

   ![](https://raw.githubusercontent.com/Fi-Null/blog-pic/master/blog-pics/netty/websocket2.png)

这里简要的说明一下WebSocket握手的过程，当Web应用程序调用`new WebSocket(url)`接口时，浏览器就会开始与对应URL地址的WebSocket服务器建立握手的连接。具体的过程是这样的：

1. 首先，浏览器与WebSocket服务器之间通过TCP的三次握手建立连接，如果连接建立失败则后续流程将不再执行，此时Web应用程序将会收到错误消息通知。
2. 当TCP连接建立成功后，浏览器会通过HTTP发送WebSocket所支持的版本号、协议的字版本号、原始地址、主机地址等一系列字段给WebSocket服务器。

```java
GET /chat HTTP/1.1  
Host: server.example.com  
Upgrade: websocket  
Connection: Upgrade  
Sec-WebSocket-Key:dGhlIHNhbXBsZSBub25jZQ==  
Origin: http://example.com  
Sec-WebSocket-Protocol: chat,superchat  
Sec-WebSocket-Version: 13  
```

这里需要重点关注的是`Sec-WebSocket-Key`这个字段，它又称为“梦幻字符串”也是一个密钥，其值采用`base64`编码的随机16字节长的字符序列，通过这个密钥服务器才能解码辨认是否为WebSocket握手请求，如果比对辨认成功则认为此协议是WebSocket协议，否则则认为是普通的HTTP协议。

3. 当WebSocket服务器接收到浏览器发送过来的握手请求后，如果数据包的数据以及格式正确、客户端和服务器的协议版本号匹配的话，就会接受本次握手连接，并给出相应的数据回复，同时回复的数据包也会采用HTTP协议进行传输。

例如：握手响应

```java
HTTP/1.1 101 Switching Protocols  
Upgrade: websocket  
Connection: Upgrade  
Sec-WebSocket-Accept:s3pPLMBiTxaQ9kYGzzhZRbK+xOo=  
Sec-WebSocket-Protocol: chat  
```

在响应头中同样存在的一个“梦幻字段”，不过它的名字叫做`Sec-WebSocket-Accept`，同样也是一个密钥，不同的是这个字符串是要让客户端辨认，当客户端拿到自动解码后，会辨认是否是一个WebSocket握手响应。

4. 当浏览器接收到WebSocket服务器回复的数据包后，如果数据包内容、格式正确的话，就表示本次连接建立成功，浏览器会触发`onopen`消息，此时Web开发人员就可以在此通过WebSocket接口中的`send`方法向WebSocket服务器发送数据了。否则握手建立失败，Web应用程序将收到`onerror`的消息，并能够知道握手连接失败的原因。

简单来说WebSocket的操作流程是：客户端首先向服务器发起一次特殊的HTTP请求，服务器接收后开始辨认请求头如果是客户端的请求则开始进行普通的TCP三次握手建立建立，否则将会按照普通的HTTP请求进行处理。

WebSocket提供了两种数据传输，一种是文本格式的数据，另一种则是二进制格式的数据。

## WebSocket与HTTP和TCP有什么关系呢？

了解完WebSocket协议的工作原理后，需要弄清楚一点的是WebSocket与TCP和HTTP之间的关系是什么样子的呢？

WebSocket与HTTP协议一样都是基于TCP的，所以它们都是可靠的协议，Web开发者调用WebSocket的`send`方法，在浏览器的实现最终都是通过TCP的接口进行传输的。

WebSocket和HTTP协议一样都属于应用层的协议，WebSocket在建立握手连接时，数据是通过HTTP协议传输的，因此会采用一部分HTTP的数据包的字段。但是在建立连接之后，真正的数据传输阶段就不需要HTTP参与了。

![](https://raw.githubusercontent.com/Fi-Null/blog-pic/master/blog-pics/netty/websocket3.png)

## http和websocket的长连接区别

HTTP1.1通过使用Connection:keep-alive进行长连接，HTTP 1.1默认进行持久连接。在一次 TCP 连接中可以完成多个 HTTP 请求，但是对每个请求仍然要单独发 header，Keep-Alive不会永久保持连接，它有一个保持时间，可以在不同的服务器软件（如Apache）中设定这个时间。这种长连接是一种“伪链接”

websocket的长连接，是一个真的全双工。长连接第一次tcp链路建立之后，后续数据可以双方都进行发送，不需要发送请求头。

keep-alive双方并没有建立真正的连接会话，服务端可以在任何一次请求完成后关闭。WebSocket 它本身就规定了是正真的、双工的长连接，两边都必须要维持住连接的状态。

## WebSocket优点

- 较少的控制开销。在连接创建后，服务器和客户端之间交换数据时，用于协议控制的数据包头部相对较小。在不包含扩展的情况下，对于服务器到客户端的内容，此头部大小只有2至10字节（和数据包长度有关）；对于客户端到服务器的内容，此头部还需要加上额外的4字节的掩码。相对于HTTP请求每次都要携带完整的头部，此项开销显著减少了。

- 更强的实时性。由于协议是全双工的，所以服务器可以随时主动给客户端下发数据。相对于HTTP请求需要等待客户端发起请求服务端才能响应，延迟明显更少；即使是和Comet等类似的长轮询比较，其也能在短时间内更多次地传递数据。

- 保持连接状态。于HTTP不同的是，Websocket需要先创建连接，这就使得其成为一种有状态的协议，之后通信时可以省略部分状态信息。而HTTP请求可能需要在每个请求都携带状态信息（如身份认证等）。

- 更好的二进制支持。Websocket定义了二进制帧，相对HTTP，可以更轻松地处理二进制内容。

- 可以支持扩展。Websocket定义了扩展，用户可以扩展协议、实现部分自定义的子协议。如部分浏览器支持压缩等。

- 更好的压缩效果。相对于HTTP压缩，Websocket在适当的扩展支持下，可以沿用之前内容的上下文，在传递类似的数据时，可以显著地提高压缩率。