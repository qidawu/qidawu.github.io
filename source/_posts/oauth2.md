---
title: OAuth 2.0 几种授权类型总结
date: 2018-10-02 22:16:13
updated:
tags: 安全
---

OAuth 协议解决了以下问题：

- 密码泄露风险
- 无法控制授权范围、有效期

OAuth 协议中，术语“授权类型（grant type）”是指应用获取访问令牌（access token）的方式。OAuth 2.0 定义了以下几种授权类型（[Grant Types](https://oauth.net/2/grant-types/)）：

- 授权码模式（[Authorization Code](https://oauth.net/2/grant-types/authorization-code/)）
- 简化模式（[Implicit](https://oauth.net/2/grant-types/implicit/)），不建议使用
- 密码模式（[Password](https://oauth.net/2/grant-types/password/)）
- 客户端模式（[Client Credentials](https://oauth.net/2/grant-types/client-credentials/)），也叫商户模式接入
- 设备码模式（[Device Code](https://oauth.net/2/grant-types/device-code/)）
- 令牌刷新模式（[Refresh Token](https://oauth.net/2/grant-types/refresh-token/)）

几种授权类型都有其对应的使用场景，各有利弊，但目的都是为了获取访问令牌，请求所需资源。访问令牌是一个用于访问用户已授权资源的临时凭据，可以将资源分为两种：

* 用户类资源：需要用户先授权。授权类型：授权码模式、密码模式、设备码模式；
* 商户类资源：无需用户授权。授权类型：客户端模式。

商户在接入认证服务器之前，需要先申请一套专用的 `client_id`、`client_secret`，据此再申请 `access_token`。下表总结了其中三种主流授权类型下，申请 `access_token` 令牌的前置条件：

| 授权方式   | grant_type           | 授权的前置条件   | 描述                                                         |
| ---------- | -------------------- | ---------------- | ------------------------------------------------------------ |
| 授权码模式 | `authorization_code` | 授权码           | 这种模式是最常见、功能最完整、流程最严密的授权模式，第三方应用需要先获取**授权码**，才能申请到令牌。它的特点就是通过第三方应用的后台服务器，与“服务提供商”的认证服务器进行互动，通过授权码换令牌，**第三方应用不接触用户密码，安全性高**。 |
| 密码模式   | `password`           | 用户的账号、密码 | 这种模式通常用在用户对该应用高度信任的情况下，或者所有服务都由同一家公司提供。在这种模式下，用户必须把自己的**用户名和密码**发给应用。应用使用这些信息，再向“服务提供商”索要授权，其风险在于应用获知了密码。但应用无需存储密码，而是存储和使用令牌即可。<br/>密码模式还有一种主流的变种，即使用**用户手机号和短信验证码**申请令牌，相比密码模式会更安全些。为了区分，可以自定义一个 `grant_type` 为 `sms_verify_code`。 |
| 客户端模式 | `client_credentials` | 无               | 第三方应用以自己的名义，而不是以用户的名义，向“服务提供商”进行认证，并获取商户类资源，而不是用户类资源。 |

# 授权码模式

OAuth 旨在让用户能够对第三方应用授予有限的访问权限。第三方应用首先需要确定所需的权限，然后将用户导向浏览器以获得其授权。简单回顾下：

![OAuth2 Authorization Code Flow](/img/security/oauth2.webp)

要开始授权流程，第三方应用需要先构建 URL。这里附一张流程图，详细总结下授权码模式的整个流程：

![OAuth 2.0 授权码模式](/img/security/oauth2.png)

报文如下：

![OAuth 2.0 授权码模式](/img/security/grant_type_of_authorization_code.png)

交互方式上特别注意浏览器会进行几次 302 重定向。流程总结如下：

1. 第三方应用首先向服务提供商申请 `client_id` 应用唯一标识、`client_secret` 密钥，用于后续获取令牌时提供身份校验；
2. 获取授权码：此时要提供预分配好的 `client_id` 标识来源，提供 `scope` 标识要申请的权限，提供 `redirect_uri` 标识授权完毕后要回跳的第三方应用的链接。
3. 第一次 302 重定向：认证服务器展示登录授权页。
4. 第二次 302 重定向：在用户提交授权，认证服务器认证成功后，会分配授权码 `code`，并重定向回第三方应用的 `redirect_uri` 。
5. 建议第三方应用要根据当前用户会话生成随机且唯一的 `state` 参数，并在接收到授权码时先进行校验，避免 CSRF 攻击。
6. 最后，第三方应用会向认证服务器申请令牌 `access_token`，此时要提供预分配好的 `code`、`client_id`、`client_secret` 以便认证。这一步是在后端之间完成的，对用户不可见。
7. `access_token` 是有有效期的，过期后需要刷新。
8. 拿到令牌 `access_token` 后，第三方应用就可以访问资源方，获取所需资源。`access_token` 相当于用户的 session id。

# 密码模式

报文如下：

![OAuth 2.0 密码模式](/img/security/grant_type_of_password.png)

# 客户端模式

客户端授权模式用于请求商户资源，而不是用户资源，报文如下：

![OAuth 2.0 客户端模式](/img/security/grant_type_of_client_credentials.png)

以微信公众平台为例，商户在完成接入并[获取 access_token](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140183) 之后，可用于如下场景：

* 自定义菜单的配置
* 消息推送
* 素材管理
* 用户管理
* 帐户管理
* 数据统计
* …… 大量服务

# 参考

https://oauth.net/2/

[What the Heck is OAuth?](https://developer.okta.com/blog/2017/06/21/what-the-heck-is-oauth)

https://developer.linkedin.com/zh-cn/docs/oauth2

http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html

[移花接木：针对OAuth2的CSRF攻击](https://www.jianshu.com/p/c7c8f51713b6)

[微信网页授权（授权码模式）](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140842)

