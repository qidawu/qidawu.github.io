---
title: Jakarta EE 系列（一）Web 技术的相关规范总结
date: 2020-10-01 23:32:08
updated:
tags: [Java]
typora-root-url: ..
---

| Specifications                                               | Description                                                 | Compatible Implementations                                   |
| ------------------------------------------------------------ | ----------------------------------------------------------- | ------------------------------------------------------------ |
| [Jakarta Servlet](https://jakarta.ee/specifications/servlet/) | A server-side API for handling HTTP requests and responses  | [Eclipse GlassFish](https://eclipse-ee4j.github.io/glassfish/) |
| [Jakarta Server Pages (JSP)](https://jakarta.ee/specifications/pages/) | Defines a template engine for web applications              | [Eclipse GlassFish](https://eclipse-ee4j.github.io/glassfish/) |
| [Jakarta Standard Tag Library (JSTL)](https://jakarta.ee/specifications/tags/) | Provides a set of tags to simplify the JSP development      | [Eclipse GlassFish](https://eclipse-ee4j.github.io/glassfish/) |
| [Jakarta Expression Language (EL)](https://jakarta.ee/specifications/expression-language/) | Defines an expression language for Java applications        | [Eclipse Expression Language](https://eclipse-ee4j.github.io/el-ri/) |
| [Jakarta Server Faces (JSF)](https://jakarta.ee/specifications/faces/) | MVC framework for building user interfaces for web apps     | [Eclipse Mojarra](https://eclipse-ee4j.github.io/mojarra/)   |
| [Jakarta MVC](https://jakarta.ee/specifications/mvc/)        | Standardizes the action-based model-view-controller pattern | [Eclipse Krazo](https://eclipse-ee4j.github.io/krazo/)       |
| [Jakarta RESTful Web Services (JAX-RS)](https://jakarta.ee/specifications/restful-ws/) | API to develop web services following the REST pattern      | [Eclipse Jersey](https://eclipse-ee4j.github.io/jersey/)     |
| [Jakarta Enterprise Web Services](https://jakarta.ee/specifications/enterprise-ws/) | Web Services for Jakarta EE architecture                    | [Eclipse GlassFish](https://eclipse-ee4j.github.io/glassfish/) |
| [Jakarta WebSocket](https://jakarta.ee/specifications/websocket/) | API for Server and Client Endpoints for WebSocket protocol  | [Eclipse Tyrus](https://eclipse-ee4j.github.io/tyrus/)       |
| ......                                                       |                                                             |                                                              |

一点关于 EL 的历史：

> Expression Language (EL) 最初受到 ECMAScript 和 XPath 表达式语言的启发。在其成立之初，参与的专家非常不愿意设计另一种表达语言，并试图使用这些语言中的每一种，但他们在不同的领域都有所欠缺。
>
> 因此，JSP Standard Tag Library (JSTL) version 1.0 (based on JSP 1.2) 首先引入了一种表达式语言，使前端页面开发者可以轻松访问和操作应用程序数据，而无需掌握如 Java、JavaScript 等编程语言相关的复杂性。
>
> 鉴于其成功，EL 随后被移入 JSP 规范（JSP 2.0/JSTL 1.1），使其在 JSP 页面中普遍可用（而不仅仅用于 JSTL 标记库的属性）。
>
> JavaServer Faces 1.0 定义了用于构建用户界面组件的标准框架，并且构建在 JSP 1.2 技术之上。由于 JSP 1.2 技术没有集成的表达语言，并且 JSP 2.0 EL 不能满足 Faces 的所有需求，因此为 Faces 1.0 开发了一个 EL 变体。Faces 专家组试图使该语言尽可能与 JSP 2.0 兼容，但是还是有一些区别。
>
> 显然需要一种单一、统一的表达语言来满足各种 Web 层技术的需求。因此，Faces 和 JSP 专家组共同制定了统一表达式语言的规范，该规范在 JSR 245 中定义，并在 JSP 2.1 和 Faces 1.2 版本中生效。
>
> JSP/JSTL/Faces 专家组也意识到 EL 的用处超出了他们自己的规范。3.0 规范是第一个将表达式语言定义为独立规范的 JSR，不依赖于其它技术。

# 参考

[JSP 标签总结](/2015/05/01/java-jsp/)

[JSP 标准标签库（JSTL）总结](/2015/05/02/java-jstl/)

[JSP EL 表达式总结](/2015/05/03/java-el/)
