---
title: Spring RestTemplate 总结
date: 2017-07-15 22:30:34
updated:
tags: [Java, Spring]
typora-root-url: ..
---

# URI 构造

要轻松地操作 URL，可以使用 Spring 的 `org.springframework.web.util.UriComponentsBuilder` 类。相比手动拼接字符串更易维护，还可以处理 URL 编码：

```java
    private URI getUri(Object body) {
        String rootUrl = "";
        String path = "";
        return UriComponentsBuilder
                .fromHttpUrl(rootUrl)
                .path(path)
                // Append the given query parameter to the existing query parameters.
                // .queryParam("key", "value1", "value2", "value3")
                // Add the given query parameters.
                .queryParams(GenericUtils.toMultiValueMap(body))  // 反射递归遍历对象的字段值，并转成 org.springframework.util.MultiValueMap
                .build()
                .encode()
                .toUri();
    }
```

`java.net.URI#toURL()` 可以将 URI 转为 `java.net.URL`。 

# HTTP 请求

构造出 `java.net.URI` 之后，可以使用 `org.springframework.web.client.RestTemplate` 如下：

```java
import lombok.extern.slf4j.Slf4j;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.*;
import org.springframework.web.client.HttpClientErrorException;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.util.UriComponentsBuilder;

import java.net.URI;

@Slf4j
public class HttpApiService {

    @Autowired
    private RestTemplate restTemplate;

    public void request(HttpMethod httpMethod, Object body) {
        try {
            URI uri;
            HttpHeaders requestHeaders = getHttpHeaders();
            HttpEntity<Object> httpEntity;

            // GET 请求，请求参数作为 query parameter 放到 url
            if (httpMethod == HttpMethod.GET) {
                uri = getUri(body);
                // 请求参数不能放到 HttpEntity，因为 GET 请求的话不会带上 request body（因为底层 HttpURLConnection#setDoOutput(false)）
                httpEntity = new HttpEntity<>(requestHeaders);
            }
            // POST 请求，请求参数作为 request body 而不放到 url
            else {
                uri = getUri(null);
                // POST 表单，request body 类型必须为 MultiValueMap：
                //   Writing [{key=[value]}] with org.springframework.http.converter.support.AllEncompassingFormHttpMessageConverter
                //   MultiValueMap 参数最终会转为形如：key1=value1&key2=value2&...
                // POST JSON：
                //   Writing [ReqDTO(key=value)] with org.springframework.http.converter.json.MappingJackson2HttpMessageConverter
                //   对方 Controller 方法入参需标注 @RequestBody，以接收整个 request body
                httpEntity = new HttpEntity<>(body, requestHeaders);
            }

            ResponseEntity<String> response = restTemplate.exchange(uri, httpMethod, httpEntity, String.class);
            if (HttpStatus.OK.equals(response.getStatusCode())) {
                log.info(response.getBody());
            }
        } catch (ResourceAccessException e) {
            log.error("TCP 连接建立失败", e);
        } catch (HttpClientErrorException | HttpServerErrorException e) {
            log.error("HTTP 请求失败，状态码：{}", e.getRawStatusCode(), e);
        }
    }

    private HttpHeaders getHttpHeaders() {
        HttpHeaders requestHeaders = new HttpHeaders();
        requestHeaders.add("Accept", MediaType.APPLICATION_JSON_VALUE);
        return requestHeaders;
    }

}
```

## CURL 形式

GET 请求：

```bash
curl --location --request GET 'http://rootUrl/path?key=value' \
--header 'Content-Type: application/x-www-form-urlencoded'
```

POST 表单（`AllEncompassingFormHttpMessageConverter`）：

```bash
curl --location --request POST 'http://rootUrl/path' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'key=value'
```

POST JSON（`MappingJackson2HttpMessageConverter`）：

```bash
curl --location --request POST 'http://rootUrl/path' \
--header 'Content-Type: application/json' \
--data-raw '{
    "key": "value"
}'
```

## 核心类解析

![RestTemplate UML](/img/spring/resttemplate/RestTemplate_UML.png)

涉及的核心类如下：

### org.springframework.*

[`org.springframework.web.client.RestTemplate`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html)

> Synchronous client to perform HTTP requests, exposing a simple, template method API over underlying HTTP client libraries such as the JDK `HttpURLConnection`, Apache HttpComponents, and others.
>
> The RestTemplate offers templates for common scenarios by HTTP method, in addition to the generalized `exchange` and `execute` methods that support of less frequent cases.

[`org.springframework.http.converter.HttpMessageConverter`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/converter/HttpMessageConverter.html)

> Strategy interface for converting from and to HTTP requests and responses.

### java.net.*

[`java.net.URL`](https://docs.oracle.com/javase/8/docs/api/java/net/URL.html)

> Class `URL` represents a Uniform Resource Locator, a pointer to a "resource" on the World Wide Web. A resource can be something as simple as a file or a directory, or it can be a reference to a more complicated object, such as a query to a database or to a search engine. More information on the types of URLs and their formats can be found at: [Types of URL](http://web.archive.org/web/20051219043731/http://archive.ncsa.uiuc.edu/SDG/Software/Mosaic/Demo/url-primer.html)

[`java.net.URLConnection`](https://docs.oracle.com/javase/8/docs/api/java/net/URLConnection.html)

> The abstract class `URLConnection` is the superclass of all classes that represent a communications link between the application and a `URL`. Instances of this class can be used both to read from and to write to the resource referenced by the `URL`.

`URLConnection` 的继承结构如下：

![URLConnection](/img/java/net/URLConnection.png)

[`java.net.HttpURLConnection`](https://docs.oracle.com/javase/8/docs/api/java/net/HttpURLConnection.html)

> A `URLConnection` with support for HTTP-specific features. See [the spec](http://www.w3.org/pub/WWW/Protocols/) for details.



[`java.net.Socket`](https://docs.oracle.com/javase/8/docs/api/java/net/Socket.html)

> This class implements client sockets (also called just "sockets"). A socket is an endpoint for communication between two machines.
> 
> The actual work of the socket is performed by an instance of the `SocketImpl` class. An application, by changing the socket factory that creates the socket implementation, can configure itself to create sockets appropriate to the local firewall.

[`java.net.SocketImpl`](https://docs.oracle.com/javase/8/docs/api/java/net/SocketImpl.html)

> The abstract class `SocketImpl` is a common superclass of all classes that actually implement sockets. It is used to create both client and server sockets.

当通过 `Socket` 类的默认无参构造方法 `new Socket()` 创建 socket 对象时，其底层实现如下图。从下图可见，将会创建抽象类 `SocketImpl` 的默认实现类 `SocksSocketImpl`：

![Default implementation of SocketImpl](/img/java/net/SocketImpl.png)

而 `SocksSocketImpl` 的继承结构如下。

![](/img/java/net/Socket.png)

`Socket` 类提供了 `getOutputStream()`、`getInputStream()` 方法，其底层实现获取 `AbstractPlainSocketImpl` 的两个私有成员变量，如下：

* `java.net.SocketInputSteram`，核心方法：

  ![SocketInputStream#socketRead0](/img/java/net/SocketInputStream_socketRead0.png)

* `java.net.SocketOutputStream`，核心方法：

  ![SocketOutputStream#socketWrite0](/img/java/net/SocketOutputStream_socketWrite0.png)

  这两个类的继承结构如下：


![java.net.SocketInputSteram & java.net.SocketOutputStream](/img/java/net/SocketInputSteram_SocketOutputStream.png)

### java.io.*

[`java.io.FileDescriptor`](https://docs.oracle.com/javase/8/docs/api/java/io/FileDescriptor.html)

> Instances of the file descriptor class serve as an opaque handle to the underlying machine-specific structure representing an open file, an open socket, or another source or sink of bytes. The main practical use for a file descriptor is to create a `FileInputStream` or `FileOutputStream` to contain it.
>
> Applications should not create their own file descriptors.

# 常见异常

下面介绍网络编程时，常见的 Socket、HTTP 异常。详见：https://docs.oracle.com/javase/8/docs/api/java/net/package-summary.html

## IOException

> In case the TCP handshakes are not complete, the connection remains unsuccessful. Consequently, **the program throws an `IOException` indicating an error occurred while establishing a new connection**.

## BindException

[`java.net.BindException`](https://docs.oracle.com/javase/8/docs/api/java/net/BindException.html)

> Signals that an error occurred while attempting to bind a socket to a local address and port.

问题：

```
java.net.BindException: Address already in use (Bind failed)
	at java.net.PlainSocketImpl.socketBind(Native Method)
	at java.net.AbstractPlainSocketImpl.bind(AbstractPlainSocketImpl.java:513)
	at java.net.ServerSocket.bind(ServerSocket.java:375)
	at java.net.ServerSocket.<init>(ServerSocket.java:237)
	at java.net.ServerSocket.<init>(ServerSocket.java:181)
...
```

原因：server-side 未成功 `bind()` 到指定端口号（如端口被其它服务占用）。

## ConnectException

[`java.net.ConnectException`](https://docs.oracle.com/javase/8/docs/api/java/net/ConnectException.html)

> Signals that an error occurred while attempting to connect a socket to a remote address and port.

### Connection refused

问题：

```
java.net.ConnectException: Connection refused (Connection refused)
	at java.net.PlainSocketImpl.socketConnect(Native Method)
	at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:476)
	at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:218)
	at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:200)
	at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:394)
	at java.net.Socket.connect(Socket.java:606)
	at java.net.Socket.connect(Socket.java:555)
	at java.net.Socket.<init>(Socket.java:451)
	at java.net.Socket.<init>(Socket.java:228)
...
```

![ResourceAccessException](/img/spring/resttemplate/ResourceAccessException.png)

原因：client-side `connect()` 建立 TCP 连接失败

* client-side `connect()` 错了服务端口号；
* server-side 服务未启动、未在 `listen()` 监听、或 `listen()` 的 `backlog` 队列数无法满足 client-side 并发连接请求数；
* 无法完成 TCP 三次握手：

![WireShark 抓包](/img/java/net/wireshark_connection_refused.png)

### Connection timed out

问题：

```
java.net.ConnectException: Connection timed out (Connection timed out)
	at java.net.PlainSocketImpl.socketConnect(Native Method)
	at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
	at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
	at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
	at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
	at java.net.Socket.connect(Socket.java:589)
	at java.net.Socket.connect(Socket.java:538)
	at java.net.Socket.<init>(Socket.java:434)
	at java.net.Socket.<init>(Socket.java:211)
...
```

原因：client-side `connect()` 超时：

```java
Socket socket = new Socket(); 
SocketAddress socketAddress = new InetSocketAddress(host, port); 
socket.connect(socketAddress, 5 * 1000);  // timeout for connect()
```

![Socket#connect 方法](/img/java/net/Socket_connect.png)

连接超时原因：client-side 发出 `sync` 包之后，server-side 未在指定时间内回复 `ack` 导致的。没有回复 `ack` 的原因可能是网络丢包、防火墙阻止服务端返回 `syn` 的 `ack` 包等。

* 防火墙原因

  > Sometimes, **firewalls block certain ports due to security reasons**. As a result, a “connection timed out” error can occur when a client is trying to establish a connection to a server. Therefore, **we should check the firewall settings** to see if it's blocking a port before binding it to a service.

## SocketTimeoutException

[`java.net.SocketTimeoutException`](https://docs.oracle.com/javase/8/docs/api/java/net/SocketTimeoutException.html)

> Signals that a timeout has occurred on a socket `accept()` or `read()`.

### Accept timed out

问题：

```
java.net.SocketTimeoutException: Accept timed out
	at java.net.PlainSocketImpl.socketAccept(Native Method)
	at java.net.AbstractPlainSocketImpl.accept(AbstractPlainSocketImpl.java:535)
	at java.net.ServerSocket.implAccept(ServerSocket.java:545)
	at java.net.ServerSocket.accept(ServerSocket.java:513)
...
```

原因：server-side `accept()` 超时：

```java
ServerSocket serverSocket = new ServerSocket(port, backlog);
serverSocket.setSoTimeout(5 * 1000);  // timeout for accept()
```

![setSoTimeout 方法](/img/java/net/ServerSocket_setSoTimeout.png)

### Read timed out

问题：

```
java.net.SocketTimeoutException: Read timed out
	at java.net.SocketInputStream.socketRead0(Native Method)
	at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
	at java.net.SocketInputStream.read(SocketInputStream.java:171)
	at java.net.SocketInputStream.read(SocketInputStream.java:141)
	at java.net.SocketInputStream.read(SocketInputStream.java:127)
...
```

原因：server-side / client-side `read()` 超时：

```java
// server-side
Socket socket = serverSocket.accept();
socket.setSoTimeout(5 * 1000);  // timeout for read()

// client-side
Socket socket = new Socket(host, port);
socket.setSoTimeout(5 * 1000);  // timeout for read()
```

![setSoTimeout 方法](/img/java/net/Socket_setSoTimeout.png)

## SocketException

[`java.net.SocketException`](https://docs.oracle.com/javase/8/docs/api/java/net/SocketException.html)

> Thrown to indicate that there is an error creating or accessing a Socket.

### Connection reset by peer

```
java.net.SocketException: Connection reset by peer (connect failed)
	at java.net.PlainSocketImpl.socketConnect(Native Method)
	at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:476)
	at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:218)
	at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:200)
	at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:394)
	at java.net.Socket.connect(Socket.java:606)
...
```

### Bad file descriptor

```
java.net.SocketException: Bad file descriptor (Write failed)
	at java.net.SocketOutputStream.socketWrite0(Native Method)
	at java.net.SocketOutputStream.socketWrite(SocketOutputStream.java:111)
	at java.net.SocketOutputStream.write(SocketOutputStream.java:155)
...
```

### Broken pipe

```
java.net.SocketException: Broken pipe (Write failed)
	at java.net.SocketOutputStream.socketWrite0(Native Method)
	at java.net.SocketOutputStream.socketWrite(SocketOutputStream.java:111)
	at java.net.SocketOutputStream.write(SocketOutputStream.java:155)
...
```

## HttpClientErrorException

[`org.springframework.web.client.HttpClientErrorException`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/HttpClientErrorException.html)

> Exception thrown when an HTTP `4xx` is received.

![HttpClientErrorException](/img/spring/resttemplate/HttpClientErrorException.png)

## HttpServerErrorException

[`org.springframework.web.client.HttpServerErrorException`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/HttpServerErrorException.html)

> Exception thrown when an HTTP `5xx` is received.

![HttpServerErrorException](/img/spring/resttemplate/HttpServerErrorException.png)

...

# 参考

https://docs.oracle.com/javase/8/docs/technotes/guides/net/index.html

https://www.baeldung.com/a-guide-to-java-sockets

https://www.baeldung.com/java-socket-connection-read-timeout
