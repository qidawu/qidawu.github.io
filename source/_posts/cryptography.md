---
title: 几种密码学常用算法总结
date: 2018-06-01 22:36:24
updated:
tags: [Java, 安全]
typora-root-url: ..
---

https://en.wikipedia.org/wiki/Cryptography

* [Cryptographic Hash Function](https://en.wikipedia.org/wiki/Cryptographic_hash_function)
* [Symmetric-key cryptography](https://en.wikipedia.org/wiki/Symmetric-key_algorithm)
* [Public-key cryptography (Asymmetric cryptography)](https://en.wikipedia.org/wiki/Public-key_cryptography)

![encryption_algorithm](/img/security/encryption_algorithm.png)

# 加密散列函数

https://en.wikipedia.org/wiki/Cryptographic_hash_function

> Cryptographic Hash Functions are cryptographic algorithms that are ways to generate and utilize specific keys to encrypt data for either symmetric or asymmetric encryption, and such functions may be viewed as keys themselves. They take a message of any length as input, and output a short, fixed-length [hash](https://en.wikipedia.org/wiki/Hash_function), which can be used in (for example) a digital signature. For good hash functions, an attacker cannot find two messages that produce the same hash.

加密散列函数（Cryptographic Hash Function）又称为不可逆算法，使用单向散列函数实现。

加密散列函数一般用于生成消息摘要（Message Digest），可用于对数据进行签名和验签，以保证数据的：

* 完整性（Integrity）
* 真实性（Authenticity）
* 不可抵赖性（Non-repudiation）

例如下图摘要（Digest）由 `SHA-1` 生成。`SHA1` 摘要长度为 20 Bytes (160 bit)，一般用 40 位十六进制数表示（4 bit (2^4) = 1 位十六进制数，因此为 160 bit / 4 = 40 位十六进制数）：

![digest](/img/security/Digest.svg)

两种破解方式：

> Ideally, the only way to find a message that produces a given hash is to attempt a [brute-force search](https://en.wikipedia.org/wiki/Brute-force_search) of possible inputs to see if they produce a match, or use a [rainbow table](https://en.wikipedia.org/wiki/Rainbow_table) of matched hashes.

用途：

* [Verifying the integrity of messages and files](https://en.wikipedia.org/wiki/File_verification)

  > Using a [cryptographic hash](https://en.wikipedia.org/wiki/Cryptographic_hash_function) and a [chain of trust](https://en.wikipedia.org/wiki/Chain_of_trust) detects malicious changes to the file. Non-cryptographic [error-detecting codes](https://en.wikipedia.org/wiki/Error-detecting_code) such as [cyclic redundancy checks](https://en.wikipedia.org/wiki/Cyclic_redundancy_check) only prevent against *non-malicious* alterations of the file, since an intentional [spoof](https://en.wikipedia.org/wiki/Spoofing_attack) can readily be crafted to have the [colliding code](https://en.wikipedia.org/wiki/Collision_attack) value.

  * [HMAC](https://en.wikipedia.org/wiki/HMAC)

    > The MAC value protects a message's [data integrity](https://en.wikipedia.org/wiki/Data_integrity), as well as its [authenticity](https://en.wikipedia.org/wiki/Message_authentication), by allowing verifiers (who also possess the secret key) to detect any changes to the message content.

* [Signature generation and verification](https://en.wikipedia.org/wiki/Digital_signature)

  > 数字签名作为维护数据信息安全的重要方法之一，可以解决伪造、抵赖、冒充和篡改等问题。其主要作用体现在以下几个方面：
  >
  > - 防冒充（身份认证）。在数字签名中，当使用私钥签名时，如果接收方或验证方用其公钥进行验证并获通过，那么可以肯定，签名人就是拥有私钥的那个人，因为私钥只有签名人拥有，其他人不可能冒充签名者。
  > - 防伪造。其他人不能伪造对数据的签名，因为私钥只有签名者自己拥有，所以其他人不可能构造出正确的签名结果数据。
  > - 防篡改。私钥加签后的摘要和数据一起发送给接收者，一旦信息被篡改，接收者可通过计算摘要和验证签名来判断该文件无效，从而保证了文件的完整性。
  > - 防抵赖。数字签名既可以作为身份认证的依据，也可以作为签名者签名操作的证据。要防止接收者抵赖，可以在数字签名系统中要求接收者返回一个自己签名的表示收到的报文，给发送者或受信任第三方。如果接收者不返回任何消息，此次通信可终止或重新开始，签名方也没有任何损失，由此双方均不可抵赖。
  > - 保密性。手写签字的文件一旦丢失，文件信息就极可能泄露。但在网络传输中，数字签名的公钥可以用于加密报文，以保证信息机密性。

* [Password verification](https://en.wikipedia.org/wiki/Password_hashing)

* [Proof-of-work](https://en.wikipedia.org/wiki/Proof-of-work_system)

* File or data identifier

  > A message digest can also serve as a means of reliably identifying a file
  >
  > * several [source code management](https://en.wikipedia.org/wiki/Source_Code_Management) systems, including [Git](https://en.wikipedia.org/wiki/Git_(software)), [Mercurial](https://en.wikipedia.org/wiki/Mercurial_(software)) and [Monotone](https://en.wikipedia.org/wiki/Monotone_(software)), use the [sha1sum](https://en.wikipedia.org/wiki/Sha1sum) of various types of content (file content, directory trees, ancestry information, etc.) to uniquely identify them. 
  >
  > * Hashes are used to identify files on [peer-to-peer](https://en.wikipedia.org/wiki/Peer-to-peer) [filesharing](https://en.wikipedia.org/wiki/Filesharing) networks. 
  >
  >   * For example, in an [ed2k link](https://en.wikipedia.org/wiki/Ed2k_link), an [MD4](https://en.wikipedia.org/wiki/MD4)-variant hash is combined with the file size, providing sufficient information for locating file sources, downloading the file, and verifying its contents. 
  >
  >   * [Magnet links](https://en.wikipedia.org/wiki/Magnet_URI_scheme) are another example. Such file hashes are often the top hash of a [hash list](https://en.wikipedia.org/wiki/Hash_list) or a [hash tree](https://en.wikipedia.org/wiki/Merkle_tree) which allows for additional benefits.
  >
  > One of the main applications of a [hash function](https://en.wikipedia.org/wiki/Hash_function) is to allow the fast look-up of data in a [hash table](https://en.wikipedia.org/wiki/Hash_table). 
  >
  > * Being hash functions of a particular kind, cryptographic hash functions lend themselves well to this application too.
  > * However, compared with [standard hash functions](https://en.wikipedia.org/wiki/Hash_function), [cryptographic hash functions](https://en.wikipedia.org/wiki/Cryptographic_hash_function) tend to be **much more expensive computationally**. For this reason, they tend to be used in contexts where it is necessary for users to protect themselves against the possibility of forgery (the creation of data with the same digest as the expected data) by potentially malicious participants.

## MD

Computes and checks [MD2](https://en.wikipedia.org/wiki/MD2_(hash_function))/[MD4](https://en.wikipedia.org/wiki/MD4)/[MD5](https://en.wikipedia.org/wiki/MD5) message digest

Java 使用例子：https://www.baeldung.com/java-md5

## SHA

Computes and checks [SHA-1](https://en.wikipedia.org/wiki/SHA-1)/[SHA-2](https://en.wikipedia.org/wiki/SHA-2) message digests

Java 使用例子：https://www.baeldung.com/sha-256-hashing-java

# 对称加密算法

https://en.wikipedia.org/wiki/Symmetric-key_algorithm

![Symmetric key encryption](/img/security/Symmetric_key_encryption.svg)

## 3DES

Java 使用例子：https://www.baeldung.com/java-3des

## AES

Java 使用例子：https://www.baeldung.com/java-secure-aes-key

Java 使用例子：https://www.baeldung.com/java-aes-encryption-decryption

# 非对称加密算法

https://en.wikipedia.org/wiki/Public-key_cryptography

公钥加密，私钥解密：

![Public key encryption](/img/security/Public_key_encryption.svg)

私钥签名，公钥验签：

![Private key signing](/img/security/Private_key_signing.svg)

![sign](/img/security/sign.png)

## RSA

Java 使用例子：https://www.baeldung.com/java-rsa

