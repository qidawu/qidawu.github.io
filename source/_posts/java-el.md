title: JSP EL 表达式总结
date: 2015-05-03 09:29:06
updated:
tags: Java
---

在 JSP 标签中指定一个属性值 `value` 时，可以使用字符串：

```
<jsp:setProperty name="box" property="perimeter" value="100"/>
```

也可以使用 EL 表达式：

```
<jsp:setProperty name="box" property="perimeter" value="${expr}"/>
```

那么，EL 表达式 `${expr}` 中可以放些什么呢？

# 操作符

EL 表达式支持大部分 Java 所提供的算术和逻辑操作符：

## 基本操作符

|操作符|描述|
|---|---|
|`.`|访问一个 Bean 的属性或者一个映射条目|
|`[]`|访问一个数组或者链表的元素|
|`()`|组织一个子表达式以改变优先级|

## 算术操作符

|操作符|操作符|描述|
|---|---|---|
|`+`||加|
|`-`||减或负|
|`*`||乘|
|`/`|`div`|除|
|`%`|`mod`|取模|

## 逻辑操作符

|操作符|操作符|描述|
|---|---|---|
|`==`|`eq`|测试是否相等|
|`!=`|`ne`|测试是否不等|
|`<`|`lt`|测试是否小于|
|`>`|`gt`|测试是否大于|
|`<=`|`le`|测试是否小于等于|
|`>=`|`ge`|测试是否大于等于|
|`&&`|`and`|测试逻辑与|
|<code>&#124;&#124;</code>|`or`|测试逻辑或|
|`!`|`not`|测试取反|
||`empty`|测试是否空值|

# 隐式对象

JSP 隐式对象（也称为预定义变量）是 JSP 容器为每个页面提供的 Java 对象，开发者可以直接使用它们而不用显式声明。

## 作用域

|对象|等价物|描述|
|---|---|---|
|`pageScope`|`this`|page 作用域|
|`requestScope`|`javax.servlet.http.HttpServletRequest`|request 作用域|
|`sessionScope`|`javax.servlet.http.HttpSession`|session 作用域|
|`applicationScope`|`javax.servlet.ServletContext`|application 作用域|

## HTTP 请求参数

|对象|等价物|描述|
|---|---|---|
|`param`|`request.getParameter(...)`|获取指定 HTTP 请求参数，字符串|
|`paramValues`|`request.getParameterValues()`|获取所有 HTTP 请求参数，字符串数组|

例如，要判断 HTTP 请求参数 `from` 是否为空，可以结合使用 JSTL `<c:if>` 和 EL 操作符 `not`、`empty`、EL 隐式对象 `param` 进行判断：

```
<c:if test="${not empty param.from}">
```

## HTTP 请求头

|对象|等价物|描述|
|---|---|---|
|`header`|`request.getHeader(...)`|获取指定 HTTP 请求头，字符串|
|`headerValues`|`request.getHeaders()`|获取所有 HTTP 请求头，字符串数组|

例如，获取请求来源：`${header.Referer}`。

## HTTP Cookie

|对象|等价物|描述|
|---|---|---|
|`cookie`|`request.getCookies()`|`javax.servlet.http.Cookie` 数组|

例如，获取指定 `Cookie` 的值：`${cookie.key.value}`。这段 EL 表达式会被 JSP 容器解析成：

```java
Cookie[] cookies = request.getCookies();
Cookie current = null;

for(Cookie cookie : cookies) { 
    if(cookie.getName().equals("key")) {
        current = cookie;
    }
}

if(current != null) { 
    out.print(current.getValue());
}
```

## 上下文

|对象|等价物|描述|
|---|---|---|
|`initParam`||上下文初始化参数，即 `web.xml` 的 `<context-param>`|
|`pageContext`|`javax.servlet.jsp.PageContext`|提供对 JSP 页面所有对象以及命名空间的访问|

# 函数

EL 表达式支持使用函数，其使用语法如下：

```
${ns:fn(param1, param2, ...)}
```

`ns` 指的是命名空间（namespace），`fn` 指的是函数的名称，`param1` 指的是第一个参数，`param2` 指的是第二个参数，以此类推。

例如，要获取一个字符串的长度，可以使用 JSTL 的 `length` 函数：

```
${fn:length("Get my length")}
```

更多 JSTL 函数，参考[这里](http://www.cnblog.me/2015/05/02/java-jstl/#函数标签库)。