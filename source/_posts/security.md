---
title: 安全专题分享
date: 2017-10-16 22:32:51
updated:
tags: [安全, 读书笔记]
typora-root-url: ..
---

本文总结的一些学习笔记，用于建立安全观。

# 信任域与信任边界

* 首先，安全问题的本质，是**信任**。一旦我们作为决策依据的条件被打破、被绕过，那么就会导致安全假设的前提条件不再可靠，变成一个伪命题。因此，把握住信任条件的度，使其恰到好处，正是设计安全方案的难点所在，也是安全这门学问的艺术魅力所在。
* 通过一个安全检查（过滤、净化）的过程，可以梳理未知的人或物，使其变得可信任。被划分出来的具有不同信任级别的区域，我们称为**信任域**，划分两个不同信任域之间的边界，我们称为**信任边界**。
* 因为信任关系被破坏，从而产生了安全问题。我们可以通过信任域的划分、信任边界的确定，来发现问题是在何处产生的。
* 数据从高等级的信任域流向低等级的信任域，是不需要经过安全检查的；数据从低等级的信任域流向高等级的信任域，则需要经过信任边界的安全检查。

# 安全基本三要素（CIA）

* 机密性（Confidentiality）：要求保护数据内容不能泄露，常见手段是加密。
* 完整性（Integrity）：要求保护数据内容是完整、没有被篡改的。常见手段是数字签名。
* 可用性（Availability）：要求保护资源是“随需而得”。如拒绝服务攻击 （简称DoS，Denial of Service） 破坏的是安全的可用性。
* 真实性（Authenticity）：通信双方的身份确认，确保数据来源于合法的用户。
* 不可抵赖性（Non-repudiation），防抵赖。常见手段是数字签名。



认证（Authentication）：我是谁？——身份

授权（Authorization）：我能做什么？——权利

凭证（Credentials）：依据是什么？——依据（凭证实现认证和授权的一种媒介，标记访问者的身份或权利）

# 3A 黄金法则

针对各个安全环节，可以使用 3A 黄金法则：

事前防御——认证（Authentication）

事中防御——授权（Authorization）

事后防御——审计（Audit）

| 认证、授权技术 | HTTP 请求头                                                  |
| -------------- | ------------------------------------------------------------ |
| 基本认证       | `Authorization: Basic <Base64("username:password")>`         |
| 摘要认证       | `Authorization: Digest <MD5(username, password, nonce, ...)>` |
| JWT            | `Authorization: Bearer <JWT Token>`                          |
| OAuth          | `Authorization: Bearer <Access Token>`                       |

## 认证（Authentication）

> 认证其实包括两个部分：身份识别和认证。
>
> - 身份识别其实就是在问 “你是谁？”，你会回答 “你是你”。
> - 身份认证则会问 “你是你吗？”，那你要证明 “你是你” 这个回答是合法的。
>
> 身份识别和认证通常是同时出现的一个过程。身份识别强调的是主体如何声明自己的身份，而身份认证强调的是，主体如何证明自己所声明的身份是合法的。
>
> 比如说：
>
> - 当你在使用用户名和密码登录的过程中，用户名起到身份识别的作用，而密码起到身份认证的作用；
> - 当你用指纹、人脸或者门卡等进行登入的过程中，这些过程同时包含了身份识别和认证。
>
> 认证形式可以大致分为三种。按照认证强度由弱到强排序，分别是：
>
> - 你知道什么（密码、密保问题等）；
> - 你拥有什么（门禁卡、手机验证码、安全令牌、U 盾等）；
> - 你是什么（生物特征，如指纹、人脸、虹膜等）。
> 
> ![authentication](/img/security/authentication.png)

典型技术：基本认证、摘要认证、JWT、单点登录（CAS 流程、OpenID）

### 基本认证（Basic Access Authentication）

https://en.wikipedia.org/wiki/Basic_access_authentication

> 在 HTTP 用户代理（如：网页浏览器）请求时，提供用户名和密码的一种方式。
>
> HTTP 请求头会包含 `Authorization` 字段，形式如下： `Authorization: Basic <凭证>`，该凭证是 `Base64("username:password")`。 
>
> 最初，基本认证是定义在 HTTP 1.0 规范（RFC 1945）中，后续的有关安全的信息可以在 HTTP 1.1 规范（RFC 2616）和 HTTP 认证规范（RFC 2617）中找到。于 1999 年 RFC 2617 过期，于 2015 年的 RFC 7617 重新被定义。

![Basic_Access_Authentication](/img/security/Basic_Access_Authentication.png)

### 摘要认证（Digest Access Authentication）

https://en.wikipedia.org/wiki/Digest_access_authentication

> 摘要认证是一种比基本认证更安全的认证方式：
>
> > It applies a hash function to the username and password before sending them over the network. In contrast, [basic access authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) uses the easily reversible [Base64](https://en.wikipedia.org/wiki/Base64) encoding instead of hashing, making it non-secure unless used in conjunction with [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security).
> >
> > Technically, digest access authentication is an application of [MD5](https://en.wikipedia.org/wiki/MD5) [cryptographic hashing](https://en.wikipedia.org/wiki/Cryptographic_hash) with usage of [nonce](https://en.wikipedia.org/wiki/Cryptographic_nonce) values to prevent [replay attacks](https://en.wikipedia.org/wiki/Replay_attack). It uses the [HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) protocol.
>
> 摘要认证最初由 RFC 2069 中被定义。RFC 2069 大致定义了一个传统的由服务器生成随机数（nonce）来维护安全性的摘要认证架构。 
>
> RFC 2069 随后被 RFC 2617 取代。RFC 2617 引入了一系列安全增强的选项。

![Digest_Access_Authentication](/img/security/Digest_Access_Authentication.png)

### 消息认证码（HMAC）

HMAC 也是一种摘要认证方式

![HMAC](/img/security/hmac-in-java.webp)

但相比上述两种认证方式仅保证用户的真实性（Authenticity），[HMAC](/posts/java-cryptography-api/#HMAC) 还能同时保证传输数据的：

* 完整性（Integrity）
* 真实性（Authenticity）
* 不可抵赖性（Non-repudiation）

### JSON Web Token

https://oauth.net/2/jwt/

## 授权（Authorization）

> 在确认完 “你是你” 之后，下一个需要明确的问题就是 “你能做什么”。
>
> 除了对 “你能做什么” 进行限制，授权机制还会对 “你能做多少” 进行限制。比如：
>
> - 手机流量授权了你能够使用多少的移动网络数据。
> - 我们申请签证的过程，其实就是一次申请授权的过程。

### OAuth

参考：[OAuth 2](/posts/oauth2/)

## 审计（Audit）

> 当你在授权之下完成操作后，安全需要检查一下 “你做了什么”，这个检查的过程就是审计。
>
> 当发现你做了某些异常操作时，安全还会提供你做了这些操作的 “证据”，让你无法抵赖，这个过程就是问责。

# 安全评估的四阶段

1. 资产等级划分：对资产进行等级划分，就是对数据做**等级划分**。当完成划分后，对要保护的目标数据已经有了一个大概的了解，接下来就是要划分信任域和信任边界了。

2. 威胁分析：威胁（Threat）是指可能造成危害的来源。威胁分析即把所有的威胁都找出来。可以采用头脑风暴法。或采用 **STRIDE** 等模型。

 ![STRIDE](/img/security/STRIDE.png)

3. 风险分析：风险（Risk）是指可能会出现的损失。风险公式：`Risk = Probability * Damage Potential`，即影响风险高低的因素，除了造成损失的大小外，还需要考虑到发生的可能性。可以采用 **DREAD** 等模型。

 ![DREAD](/img/security/DREAD.png)

4. 确认解决方案

# 安全方案设计的四原则

* **默认安全性原则（Secure by Default）**，最基本也是最重要的原则。即：
   * 黑、白名单。随着防火墙、ACL 技术的兴起，使得直接暴露在互联网上的系统得到了保护。比如一个网站的数据库，在没有保护的情况下，数据库服务端口是允许任何人随意连接的；在有了防火墙的保护后，通过ACL可以控制只允许信任来源的访问。这些措施在很大程度上保证了系统软件处于信任边界之内，从而杜绝了大部分的攻击来源。因此如果更多地使用白名单（如防火墙、ACL），系统就会变得更安全。
   * 最小权限原则。安全设计的基本原则之一。最小权限原则要求系统只授予主体必要的权限，而不要过度授权，这样能有效地减少系统、网络、应用、数据库出错的机会。
* **纵深防御原则 （Defense in Depth）**，其包含两层含义：
   * 首先，在各个不同层面、不同方面实施安全方案，避免出现疏漏，不同安全方案之间需要相互配合，构成一个整体；
   * 其次，在正确的地方做正确的事情，即：在解决根本问题的地方实施针对性的安全方案。
* **数据与代码分离原则**
   * 适用于各种由于“注入”而引发的安全问题，如 XSS、SQL 注入、CRLF 注入、X-Path 注入。
* **不可预测性原则（Unpredictable）**
   * 能有效地对抗基于篡改、伪造（如 CSRF）的攻击，其实现往往需用到加密算法、随机数算法、哈希算法等。

总结这几条原则：

* Secure By Default：是时刻要牢记的原则；
* 纵深防御：是要更全面、更正确地看待问题；
* 数据与代码分离：是从漏洞成因上看问题；
* 不可预测性：则是从克服攻击方法的角度看问题。

# 参考

互联网安全的核心问题，是**数据安全**的问题。

《白帽子讲 Web 安全》

https://time.geekbang.org/column/intro/262