title: Spring MVC 常用注解
date: 2016-01-13 20:32:08
updated:
tags: Java
---

# @RequestMapping

[`@RequestMapping`](http://static.springsource.org/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestMapping.html)

Annotation for mapping web requests onto specific handler classes and/or handler methods.

Handler methods which are annotated with this annotation are allowed to have very flexible signatures. They may have arguments of the following types, in arbitrary order (任意顺序):

## @PathVariable

[`@PathVariable`](http://static.springsource.org/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/PathVariable.html) 用于标注某个方法参数与某个 *URI 模板变量（URI template variable）* 的绑定关系，常用于 *RESTful URL*，例如 `/hotels/{hotel}`。

## @RequestParam

[`@RequestParam`](http://static.springsource.org/spring/docs/3.0.x/javadoc-api/org/springframework/web/bind/annotation/RequestParam.html) 用于标注某个方法参数与某个 *WEB 请求参数（web request parameter）* 的绑定关系。

使用时需要注意 `required` 这个属性：

* 方法参数不写 `@RequestParam`，默认的 `required` 为 `false`
* 方法参数写了 `@RequestParam`，默认的 `required` 为 `true`
* 方法参数同时写了 `@RequestParam` + `defaultValue`，默认的 `required` 为 `false`

## @RequestHeader

用于标注某个方法参数与某个 *WEB 请求头（web request header）* 的绑定关系。

## @RequestBody

用于标注某个方法参数与某个 *WEB 请求体（web request body）* 的绑定关系。使用 `@RequestBody` 将会使用合适的 `HttpMessageConverter` 将 *WEB 请求体（web request body）* 写入指定对象。

## @CookieValue

用于标注某个方法参数与某个 *HTTP cookie* 的绑定关系。方法参数可以是 `javax.servlet.http.Cookie`，也可以是具体的 Cookie 值（如字符串、数字类型等）。

## Request / Response

用于访问当前 `javax.servlet.http.HttpServletRequest` / `javax.servlet.http.HttpServletResponse`。

```java
@RequestMapping("/index")
public void go(HttpServletRequest request, HttpServletResponse response) {
    response.getWriter().write("hello world");
}
```

## InputStream / Reader

用于访问当前请求内容的 `java.io.InputStream` / `java.io.Reader`

## OutputStream / Writer

用于生成当前响应内容的 `java.io.OutputStream` / `java.io.Writer`

```java
@RequestMapping("/index")
public void go(Writer writer) {
    writer.write("hello world");
}
```

浏览器显示：

```
hello world
```

## Session

用于访问当前 `javax.servlet.http.HttpSession`

```java
@RequestMapping("/index")
public void go(HttpSession session) {
    System.out.println(session.getAttribute(xxx));
}
```

## Locale

用于访问当前请求的区域设置（`java.util.Locale`）。

```java
@RequestMapping("/index")
public void go(Locale locale) {
    System.out.println(locale);
}
```

## HttpEntity<?>

## Map / Model / ModelMap

用于在 `Controller` 层填充将暴露给 `View` 层的 `Model` 。

# @ResponseBody

用于标注某个方法返回值与 *WEB 响应体（response body）* 的绑定关系。使用 `@ResponseBody` 将会跳过 `ViewResolver` 部分，调用适合的 `HttpMessageConverter`，将方法返回值作为 *WEB 响应体（response body）* 写入输出流。

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