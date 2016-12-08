title: Spring MVC 常用注解
date: 2016-01-14 23:32:08
updated:
tags: Java
---

一张图简要描述 Spring MVC 的处理流程：

![Spring MVC](/img/spring-mvc/spring-mvc.png)

* Spring MVC 的核心前端控制器 `DispatcherServlet` 接收 HTTP 请求并询问 `Handler mapping` 该请求应该转发到哪个 `Controller` 方法。
* `Controller` 业务处理完毕，返回 *逻辑视图名(通常是一个字符串)* 。
* 最后 `viewResolver` 解析逻辑视图名并返回相应的 `View`,如 JSP、FreeMarker。

下面介绍编写 `Controller` 过程中常用的注解：

# @RequestMapping

[`@RequestMapping`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestMapping.html) 用于将 HTTP 请求映射到指定的 `Controller` 类或方法。标注了这个注解的方法可以拥有非常灵活的方法签名。其方法参数可以是下列任一类型。

本文将分为三类介绍：

## 常规类型

使用这类方法参数有点类似于传统的 Servlet 编程，因此称之为常规类型：

### Request / Response

用于访问当前 `javax.servlet.http.HttpServletRequest` / `javax.servlet.http.HttpServletResponse`。

```java
@RequestMapping("/index")
public void go(HttpServletRequest request, HttpServletResponse response) {
    request.getHeader("host"); // 读取指定 HTTP 请求头
    response.getWriter().write("hello world"); // 浏览器将会显示：hello world
}
```

### InputStream / Reader

用于访问当前请求内容的 `java.io.InputStream` / `java.io.Reader`

### OutputStream / Writer

用于生成当前响应内容的 `java.io.OutputStream` / `java.io.Writer`

```java
@RequestMapping("/index")
public void go(Writer writer) {
    writer.write("hello world"); // 浏览器将会显示：hello world
}
```

### Session

用于访问当前 `javax.servlet.http.HttpSession`

```java
@RequestMapping("/index")
public void go(HttpSession session) {
    session.getAttribute("xxx"); // 读取指定 Session 值
}
```

### HttpEntity<?>

[`HttpEntity<?>`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/HttpEntity.html) 用于同时访问 *HTTP 请求头和请求体（HTTP request headers and contents）* 。

```java
@RequestMapping("/index")
public void go(HttpEntity<String> httpEntity) {
    String body = httpEntity.getBody();
    HttpHeaders headers = httpEntity.getHeaders();
    String host = headers.getFirst("host");
}
```

## 注解类型

尽管使用常规类型的方法参数更接近于人们所熟悉的传统 Servlet 编程，但在 Spring 编程中却不建议这么做。因为这样会导致 JavaBean 与 Servlet 容器耦合，侵入性强，难以进行单元测试（如 Mock 测试）。最佳实践应当是传入注解后被解析好的数据类型，下面介绍这些常用的注解：

### @PathVariable

[`@PathVariable`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/PathVariable.html) 用于标注某个方法参数与某个 *URI 模板变量（URI template variable）* 的绑定关系，常用于 *RESTful URL*，例如 `/hotels/{hotel}`。

### @RequestParam

[`@RequestParam`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestParam.html) 用于标注某个方法参数与某个 *HTTP 请求参数（HTTP request parameter）* 的绑定关系。

使用时需要注意 `required` 这个属性：

* 方法参数不写 `@RequestParam`，默认的 `required` 为 `false`
* 方法参数写了 `@RequestParam`，默认的 `required` 为 `true`
* 方法参数同时写了 `@RequestParam` + `defaultValue`，默认的 `required` 为 `false`

例子：

```
POST /test HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded

data=123,234
```

```
GET /test?data=123,234 HTTP/1.1
Host: localhost:8080
```

```java
@RequestMapping("/test")
public void test(@RequestParam("data") ArrayList<String> data) {
}
```

### @RequestHeader

[`@RequestHeader`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestHeader.html) 用于标注某个方法参数与某个 *HTTP 请求头（HTTP request header）* 的绑定关系。

### @RequestBody

[`@RequestBody`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestBody.html) 用于标注某个方法参数与某个 *HTTP 请求体（HTTP request body）* 的绑定关系。 `@RequestBody` 会调用合适的 [message converters](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/converter/HttpMessageConverter.html) 将 *HTTP 请求体（HTTP request body）* 写入指定对象。

例子：

```
POST /test HTTP/1.1
Host: localhost:8080
Content-Type: application/json

["123", "234"]
```

```java
@RequestMapping("/test")
public void test(@RequestBody List<String> data) {
}
```

### @CookieValue

[`@CookieValue`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/CookieValue.html) 用于标注某个方法参数与某个 *HTTP cookie* 的绑定关系。方法参数可以是 `javax.servlet.http.Cookie`，也可以是具体的 Cookie 值（如字符串、数字类型等）。

举个例子：

```java
@ResponseBody
@RequestMapping("/index")
public Employee getEmployeeBy(
    @RequestParam("name") String name, 
    @RequestHeader("host") String host, 
    @RequestBody String body) {...}
```

## 其它类型

### Map / Model / ModelMap

`Map` / [`Model`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/ui/Model.html) / [`ModelMap`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/ui/ModelMap.html) 用于在 `Controller` 层填充将会暴露给 `View` 层的 `Model` 对象。

# @ResponseBody

用于标注某个方法返回值与 *WEB 响应体（response body）* 的绑定关系。 `@ResponseBody` 会跳过 `ViewResolver` 部分，调用合适的 [message converters](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/converter/HttpMessageConverter.html)，将方法返回值作为 *WEB 响应体（response body）* 写入输出流。

```java
@RequestMapping("/index")
@ResponseBody
public Date go() {
    return new Date();
}
```

# @ResponseStatus

[`@ResponseStatus`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/ResponseStatus.html) 用于返回 HTTP 响应码，例如返回 404：

```java
@RequestMapping("/index")
@ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "找不到网页")
public void go() {
}
```