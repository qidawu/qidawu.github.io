---
title: Java 加密篇之算法总结
date: 2018-06-01 22:36:24
updated:
tags: [Java, 安全]
typora-root-url: ..
---

![encryption_algorithm](/img/security/encryption_algorithm.png)

# 单向加密

单向加密又称为不可逆加密算法，其密钥是由加密散列函数生成的。单向散列函数一般用于产生消息摘要，密钥加密等。

例如下图摘要由 `SHA-1` 生成。`SHA1` 摘要长度为 20 Bytes (160 bit)，一般用 40 位十六进制数表示（1 Byte = 8 bit，4 bit (2^4) = 1 位十六进制数，因此 160 bit / 4 = 40 位十六进制数表）：

![digest](/img/security/digest.png)

# 对称加密

![symmetric_encryption](/img/security/symmetric_encryption.png)

# RSA

## 非对称加密

公钥加密，私钥解密

![asymmetric_encryption](/img/security/asymmetric_encryption.png)



## 签名验签

私钥签名，公钥验签

![sign](/img/security/sign.png)