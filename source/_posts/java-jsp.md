title: "JSP 语法和标签总结"
date: 2015-05-10 21:20:37
updated: 
tags: Java
---

# JSP

JSP 有以下三类标签：

## JSP Directive

指令标签用于设置与整个 JSP 页面相关的属性，非常常用。

|标签|标签|描述|
|---|---|---|
|`<%@ page ... %>`|`<jsp:directive.page attribute="value" />`|定义页面的依赖属性，例如脚本语言、页面编码、缓存需求等等|
|`<%@ include ... %>`|`<jsp:directive.include file="relative url" />`|引入其它文件，例如 JSP、HTML、文本文件|
|`<%@ taglib ... %>`|`<jsp:directive.taglib uri="uri" prefix="prefixOfTag" />`|引入标签库，可以是 JSP 标准标签库（JSTL）、也可以是自定义标签库|

## JSP Syntax

语法标签是 Java 早期为了便于开发人员在 JSP 页面中书写业务逻辑而设计的，但目前不再建议使用。

|标签|标签|描述|
|---|---|---|
|`<% scriptlet %>`|`<jsp:scriptlet> scriptlet </jsp:scriptlet>`|脚本程序，可以包含任意有效的 Java 语句、变量、方法或表达式|
|`<%! declaration %>`|`<jsp:declaration> declaration </jsp:declaration>`|声明语句，可以声明一个或多个变量、方法，供后面的 Java 代码使用|
|`<%= expression %>`|`<jsp:expression> expression </jsp:expression>`|表达式，其结果会被转为字符串并输出到 HTML 页面|
|`<%-- comment --%>`||代码注释|

举个栗子：

```
<html>
<body>
    <%! String output = "world"; %>
    
    <% out.println("Hello " + output); %>
    <br/>
    <%= "Hello " + output %>
</body>
</html>
```

渲染输出：
```
Hello world 
Hello world
```

上例中，虽然结合使用这三种语法标签，可以在 JSP 页面中写出大段的 Java 逻辑代码，但强烈不建议这么做，因为这样会导致前端页面和业务逻辑之间紧耦合，以致后续难以维护。

## JSP Action

函数标签是一些预定义好的行为标签，但不常用。

|标签|描述|
|---|---|
|`<jsp:include />`|用于在当前页面中包含静态或动态资源|
|`<jsp:useBean />`|寻找和初始化一个 JavaBean 组件|
|`<jsp:setProperty />`|设置 JavaBean 组件的值|
|`<jsp:getProperty />`|将 JavaBean 组件的值插入到 output 中|
|`<jsp:forward />`|从一个 JSP 文件向另一个文件传递一个包含用户请求的 request 对象|
|`<jsp:plugin />`|用于在生成的 HTML 页面中包含 Applet 和 JavaBean 对象|
|`<jsp:element />`|动态创建一个 XML 元素|
|`<jsp:attribute />`|定义动态创建的 XML 元素的属性|
|`<jsp:body />`|定义动态创建的 XML 元素的主体|
|`<jsp:text />`|用于封装模板数据|

# JSTL

# EL