---
title: Spring MVC 常用注解
date: 2016-01-14 23:32:08
updated:
tags: Java
---

# DispatcherServlet 介绍

一张图简要描述 Spring MVC 的处理流程：

![Spring MVC](/img/spring/spring-mvc.png)

* Spring MVC 的核心前端控制器 `DispatcherServlet` 接收 HTTP 请求并询问 `Handler mapping` 该请求应该转发到哪个 `Controller` 方法。
* `Controller` 业务处理完毕，返回 *逻辑视图名(通常是一个字符串)* 。
* 最后 `viewResolver` 解析逻辑视图名并返回相应的 `View`,如 JSP、FreeMarker。

# Controller 实现

下面介绍编写控制器过程中常用的注解：

## 定义一个控制器

### @Controller

Traditional MVC controller relying on a [view technology](https://spring.io/understanding/view-templates) to perform server-side rendering.

### @RestController

RESTful web service controller simply populates and returns a object that will be written directly to the HTTP response as JSON. Thanks to Spring’s HTTP message converter support, you don’t need to do this conversion manually. Because [Jackson 2](http://wiki.fasterxml.com/JacksonHome) is on the classpath, Spring’s [`MappingJackson2HttpMessageConverter`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/converter/json/MappingJackson2HttpMessageConverter.html) is automatically chosen to convert the object to JSON.

It’s shorthand for `@Controller` and `@ResponseBody` rolled together. 

### @ControllerAdvice

Classes annotated with `@ControllerAdvice` can contain `@ExceptionHandler`, `@InitBinder`, and `@ModelAttribute` annotated methods, and these methods will apply to `@RequestMapping` methods across all controller hierarchies as opposed to the controller hierarchy within which they are declared.

`@RestControllerAdvice` is an alternative where `@ExceptionHandler` methods assume `@ResponseBody` semantics by default.

Both `@ControllerAdvice` and `@RestControllerAdvice` can target a subset of controllers:

```
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class AnnotationAdvice {}

// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class BasePackageAdvice {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class AssignableTypesAdvice {}
```

## 映射请求

### @RequestMapping

[`@RequestMapping`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestMapping.html) 用于将 HTTP 请求映射到指定的 `Controller` 类或方法，可匹配的请求属性如下：

#### 属性

* `value` 用于匹配指定的请求路径，例如：`value = "/index"`。
* `method` 用于匹配指定的请求方法，例如：`method = RequestMethod.POST`。
* `consumes` 用于匹配指定的请求头 `Content-Type`，例如：`consumes = MediaType.APPLICATION_JSON_UTF8_VALUE`。
* `produces` 用于匹配指定的请求头 `Accept`，例如：`produces = MediaType.APPLICATION_JSON_UTF8_VALUE`。
* `params` 用于匹配指定的请求参数，例如：
  - 匹配存在：`params = "myParam"`
  - 匹配不存在：`params = "!myParam"`
  - 匹配指定参数值：`params = "myParam=myValue"` 
* `headers` 用于匹配指定的请求头，例如：
  - 匹配存在：`headers = "myHeader"`
  - 匹配不存在：`headers = "!myHeader"`
  - 匹配指定值：`headers = "myHeader=myValue"`

尽管你可以使用媒体类型通配符（例如：`content-type=text/*` 或 `accept=xxx`）去匹配 `Content-Type` 或 `Accept` 请求头，但还是更推荐使用 `consumes` 或 `produces` 属性。因为它们专门用于此目的。

#### 变体

Spring Framework 4.3 还引入了五个等价的变体注解，相当于 `@RequestMapping` 注解与 `method` 属性的组合使用：

```
@GetMapping
@PostMapping
@PutMapping
@DeleteMapping
@PatchMapping
```

## 定义处理器方法

标注了 `@RequestMapping` 注解的方法可以拥有非常灵活的方法签名。

### 支持的方法参数类型

其方法参数可以是下列任一类型：

#### Request / Response

用于访问当前 `javax.servlet.http.HttpServletRequest` / `javax.servlet.http.HttpServletResponse`。

```java
@RequestMapping("/index")
public void go(HttpServletRequest request, HttpServletResponse response) {
    request.getHeader("host"); // 读取指定 HTTP 请求头
    response.getWriter().write("hello world"); // 浏览器将会显示：hello world
}
```

使用这类方法参数有点类似于传统的 Servlet 编程。

#### InputStream / Reader

用于访问当前请求内容的 `java.io.InputStream` / `java.io.Reader`

#### OutputStream / Writer

用于生成当前响应内容的 `java.io.OutputStream` / `java.io.Writer`

```java
@RequestMapping("/index")
public void go(Writer writer) {
    writer.write("hello world"); // 浏览器将会显示：hello world
}
```

#### Session

用于访问当前 `javax.servlet.http.HttpSession`

```java
@RequestMapping("/index")
public void go(HttpSession session) {
    session.getAttribute("xxx"); // 读取指定 Session 值
}
```

#### HttpEntity<?>

[`HttpEntity<?>`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/HttpEntity.html) 用于同时访问 *HTTP 请求头和请求体（HTTP request headers and contents）* 。

```java
@RequestMapping("/index")
public void go(HttpEntity<String> httpEntity) {
    String body = httpEntity.getBody();
    HttpHeaders headers = httpEntity.getHeaders();
    String host = headers.getFirst("host");
}
```



尽管使用常规类型的方法参数更接近于人们所熟悉的传统 Servlet 编程，但在 Spring 编程中却不建议这么做。因为这样会导致 JavaBean 与 Servlet 容器耦合，侵入性强，难以进行单元测试（如 Mock 测试）。最佳实践应当是传入注解后被解析好的数据类型，下面介绍这些常用的注解：

#### @PathVariable

[`@PathVariable`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/PathVariable.html) 用于标注某个方法参数与某个 *URI 模板变量（URI template variable）* 的绑定关系，常用于 *RESTful URL*，例如 `/hotels/{hotel}`。

#### @RequestParam

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
public void test(@RequestParam("data") String data) {
    log.info(data);  // 123, 234
}
```

Type conversion is applied automatically if the target method parameter type is not String. See the section called “[Method Parameters And Type Conversion](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-ann-typeconversion)”.

```java
@RequestMapping("/test")
public void test(@RequestParam("data") ArrayList<String> data) {
    log.info(data.toString());  // 123, 234
}
```

When an `@RequestParam` annotation is used on a `Map<String, String>` or `MultiValueMap<String, String>` argument, the map is populated with all request parameters.

```java
@RequestMapping("/test")
public void test(@RequestParam("data") Map<String, String> data) {
    log.info(data.get("data"));  // 123, 234
}
```

#### @RequestBody

[`@RequestBody`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestBody.html) 用于标注某个方法参数与某个 *HTTP 请求体（HTTP request body）* 的绑定关系。 `@RequestBody` 会调用合适的 [message converters](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/converter/HttpMessageConverter.html) 将 *HTTP 请求体（HTTP request body）* 写入指定对象，默认的 `HttpMessageConverter 如下：

* `ByteArrayHttpMessageConverter` converts byte arrays.
* `StringHttpMessageConverter` converts strings.
* `FormHttpMessageConverter` converts form data to/from a MultiValueMap<String, String>.
* `SourceHttpMessageConverter` converts to/from a javax.xml.transform.Source.

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

An `@RequestBody` method parameter can be annotated with `@Valid`, in which case it will be validated using the configured `Validator` instance. When using the MVC namespace or the MVC Java config, a JSR-303 validator is configured automatically assuming a JSR-303 implementation is available on the classpath.

#### @RequestHeader

[`@RequestHeader`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestHeader.html) 用于标注某个方法参数与某个 *HTTP 请求头（HTTP request header）* 的绑定关系。

#### @CookieValue

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

#### Map / Model / ModelMap

`Map` / [`Model`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/ui/Model.html) / [`ModelMap`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/ui/ModelMap.html) 用于在 `Controller` 层填充将会暴露给 `View` 层的 `Model` 对象。

### 支持的方法返回类型

#### HttpEntity<?>

#### ResponseEntity<?>

#### @ResponseBody

用于标注某个方法返回值与 *WEB 响应体（response body）* 的绑定关系。 `@ResponseBody` 会跳过 `ViewResolver` 部分，调用合适的 [message converters](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/converter/HttpMessageConverter.html)，将方法返回值作为 *WEB 响应体（response body）* 写入输出流。

```java
@RequestMapping("/index")
@ResponseBody
public Date go() {
    return new Date();
}
```

#### @ResponseStatus

[`@ResponseStatus`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/ResponseStatus.html) 用于返回 HTTP 响应码，例如返回 404：

```java
@RequestMapping("/index")
@ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "找不到网页")
public void go() {
}
```
# 处理视图

# 处理异常

# CORS 支持

# 核心配置

# 参考

《[Serving Web Content with Spring MVC](https://spring.io/guides/gs/serving-web-content/)》

《[Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/)》

《[Enabling Cross Origin Requests for a RESTful Web Service](https://spring.io/guides/gs/rest-service-cors/)》