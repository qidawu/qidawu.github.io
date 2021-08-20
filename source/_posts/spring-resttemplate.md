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

## 异常处理

潜在的非受检异常如下：

* Exception thrown when an I/O error occurs. 原因是 `java.net.HttpURLConnection#connect()` 建立 TCP 连接失败从而抛出 `java.net.ConnectException: Connection refused (Connection refused)`

  ![ResourceAccessException](/img/spring/resttemplate/ResourceAccessException.png)

* `java.net.SocketException: Bad file descriptor (Write failed)`

  ```java
  java.net.SocketException: Bad file descriptor (Write failed)
  	at java.net.SocketOutputStream.socketWrite0(Native Method) ~[?:1.8.0_271]
  	at java.net.SocketOutputStream.socketWrite(SocketOutputStream.java:111) ~[?:1.8.0_271]
  	at java.net.SocketOutputStream.write(SocketOutputStream.java:155) ~[?:1.8.0_271]
    ...
  ```

* Exception thrown when an HTTP 4xx is received.

  ![HttpClientErrorException](/img/spring/resttemplate/HttpClientErrorException.png)

* Exception thrown when an HTTP 5xx is received.

  ![HttpServerErrorException](/img/spring/resttemplate/HttpServerErrorException.png)

* ...

# UML 解析

![RestTemplate UML](/img/spring/resttemplate/RestTemplate_UML.png)

# 参考

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html

> Synchronous client to perform HTTP requests, exposing a simple, template method API over underlying HTTP client libraries such as the JDK `HttpURLConnection`, Apache HttpComponents, and others.
>
> The RestTemplate offers templates for common scenarios by HTTP method, in addition to the generalized `exchange` and `execute` methods that support of less frequent cases.

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/converter/HttpMessageConverter.html

> Strategy interface for converting from and to HTTP requests and responses.



https://docs.oracle.com/javase/8/docs/api/java/net/URL.html

> Class `URL` represents a Uniform Resource Locator, a pointer to a "resource" on the World Wide Web. A resource can be something as simple as a file or a directory, or it can be a reference to a more complicated object, such as a query to a database or to a search engine. More information on the types of URLs and their formats can be found at: [Types of URL](http://web.archive.org/web/20051219043731/http://archive.ncsa.uiuc.edu/SDG/Software/Mosaic/Demo/url-primer.html)

https://docs.oracle.com/javase/8/docs/api/java/net/URLConnection.html

> The abstract class `URLConnection` is the superclass of all classes that represent a communications link between the application and a `URL`. Instances of this class can be used both to read from and to write to the resource referenced by the `URL`.

`URLConnection` 的继承结构如下：

![URLConnection](/img/java/net/URLConnection.png)

https://docs.oracle.com/javase/8/docs/api/java/net/HttpURLConnection.html

> A `URLConnection` with support for HTTP-specific features. See [the spec](http://www.w3.org/pub/WWW/Protocols/) for details.



https://docs.oracle.com/javase/8/docs/api/java/net/Socket.html

> This class implements client sockets (also called just "sockets"). A socket is an endpoint for communication between two machines.
> 
> The actual work of the socket is performed by an instance of the `SocketImpl` class. An application, by changing the socket factory that creates the socket implementation, can configure itself to create sockets appropriate to the local firewall.

https://docs.oracle.com/javase/8/docs/api/java/net/SocketImpl.html

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



https://docs.oracle.com/javase/8/docs/api/java/io/FileDescriptor.html

> Instances of the file descriptor class serve as an opaque handle to the underlying machine-specific structure representing an open file, an open socket, or another source or sink of bytes. The main practical use for a file descriptor is to create a `FileInputStream` or `FileOutputStream` to contain it.
>
> Applications should not create their own file descriptors.

