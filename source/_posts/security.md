---
title: 安全专题分享
date: 2018-10-16 22:32:51
updated:
tags: [安全, 读书笔记]
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