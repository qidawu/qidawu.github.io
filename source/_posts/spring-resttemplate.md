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
                httpEntity = new HttpEntity<>(requestHeaders);
            }
            // POST 请求，请求参数作为 request body
            else {
                uri = getUri(null);
                // 如果 body 类型为 MultiValueMap：
                //   Writing [{key=[value]}] with org.springframework.http.converter.support.AllEncompassingFormHttpMessageConverter
                // 否则：
                //   Writing [ReqDTO(key=value)] with org.springframework.http.converter.json.MappingJackson2HttpMessageConverter
                //   对方 Controller 方法入参标注 @RequestBody 接收整个请求体
                httpEntity = new HttpEntity<>(body, requestHeaders);
            }

            ResponseEntity<String> response = restTemplate.exchange(uri, httpMethod, httpEntity, String.class);
            if (HttpStatus.OK.equals(response.getStatusCode())) {
                log.info(response.getBody());
            }
        } catch (HttpClientErrorException e) {
            log.error("HTTP 请求异常，状态码：{}", e.getRawStatusCode(), e);
        }
    }

    private HttpHeaders getHttpHeaders() {
        HttpHeaders requestHeaders = new HttpHeaders();
        requestHeaders.add("Accept", MediaType.APPLICATION_JSON_VALUE);
        return requestHeaders;
    }

}
```

GET 请求：

```bash
curl --location --request GET 'http://rootUrl/path?key=value' \
--header 'Content-Type: application/x-www-form-urlencoded'
```

POST 请求（`AllEncompassingFormHttpMessageConverter`）：

```bash
curl --location --request POST 'http://rootUrl/path' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'key=value'
```

POST 请求（`MappingJackson2HttpMessageConverter`）：

```bash
curl --location --request POST 'http://rootUrl/path' \
--header 'Content-Type: application/json' \
--data-raw '{
    "key": "value"
}'
```

# 异常处理

潜在的非受检异常如下：

* Exception thrown when an I/O error occurs. 原因是 `java.net.HttpURLConnection#connect()` 建立 TCP 连接失败从而抛出 `java.net.ConnectException: Connection refused (Connection refused)`

  ![ResourceAccessException](/img/spring/resttemplate/ResourceAccessException.png)

* Exception thrown when an HTTP 4xx is received.

  ![HttpClientErrorException](/img/spring/resttemplate/HttpClientErrorException.png)

* Exception thrown when an HTTP 5xx is received.

  ![HttpServerErrorException](/img/spring/resttemplate/HttpServerErrorException.png)

* ...

# 参考

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html

> Synchronous client to perform HTTP requests, exposing a simple, template method API over underlying HTTP client libraries such as the JDK `HttpURLConnection`, Apache HttpComponents, and others.
>
> The RestTemplate offers templates for common scenarios by HTTP method, in addition to the generalized `exchange` and `execute` methods that support of less frequent cases.

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/converter/HttpMessageConverter.html

> Strategy interface for converting from and to HTTP requests and responses.
