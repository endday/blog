---
title: HTTP和HTTPS协议
url: https://www.yuque.com/endday/blog/vpgd6m
---

> <https://zhuanlan.zhihu.com/p/72616216?utm_source=wechat_session&ivk_sa=1024320u>
> <https://blog.csdn.net/u014294681/article/details/86705999>

<a name="iXYii"></a>

## HTTPS

<a name="Ih1cs"></a>

### 为什么要用https？

可以看到访问的账号密码都是明文传输， 这样客户端发出的请求很容易被不法分子截取利用，因此，HTTP协议不适合传输一些敏感信息，比如：各种账号、密码等信息，使用http协议传输隐私信息非常不安全。
**一般http中存在如下问题：**

- 请求信息明文传输，容易被窃听截取。
- 数据的完整性未校验，容易被篡改
- 没有验证对方身份，存在冒充危险

<a name="CfFyX"></a>

### 什么是HTTPS?

为了解决上述HTTP存在的问题，就用到了HTTPS。
HTTPS 协议（HyperText Transfer Protocol over Secure Socket Layer）：一般理解为HTTP+SSL/TLS，通过 SSL证书来验证服务器的身份，并为浏览器和服务器之间的通信进行加密。
**那么SSL又是什么？**
SSL（Secure Socket Layer，安全套接字层）：1994年为 Netscape 所研发，SSL 协议位于 TCP/IP 协议与各种应用层协议之间，为数据通讯提供安全支持。
TLS（Transport Layer Security，传输层安全）：其前身是 SSL，它最初的几个版本（SSL 1.0、SSL 2.0、SSL 3.0）由网景公司开发，1999年从 3.1 开始被 IETF 标准化并改名，发展至今已经有 TLS 1.0、TLS 1.1、TLS 1.2 三个版本。SSL3.0和TLS1.0由于存在安全漏洞，已经很少被使用到。TLS 1.3 改动会比较大，目前还在草案阶段，目前使用最广泛的是TLS 1.1、TLS 1.2。
**SSL发展史（互联网加密通信）**

1. 1994年NetSpace公司设计SSL协议（Secure Sockets Layout）1.0版本，但未发布。
2. 1995年NetSpace发布SSL/2.0版本，很快发现有严重漏洞
3. 1996年发布SSL/3.0版本，得到大规模应用
4. 1999年，发布了SSL升级版TLS/1.0版本，目前应用最广泛的版本
5. 2006年和2008年，发布了TLS/1.1版本和TLS/1.2版本

HTTPS数据传输流程
![image.png](..\assets\vpgd6m\1647914391789-5e881c98-6b89-4186-ab83-a324aa5088b3.png)

1. 首先客户端通过URL访问服务器建立SSL连接。
2. 服务端收到客户端请求后，会将网站支持的证书信息（证书中包含公钥）传送一份给客户端。
3. 客户端的服务器开始协商SSL连接的安全等级，也就是信息加密的等级。
4. 客户端的浏览器根据双方同意的安全等级，建立会话密钥，然后利用网站的公钥将会话密钥加密，并传送给网站。
5. 服务器利用自己的私钥解密出会话密钥。
6. 服务器利用会话密钥加密与客户端之间的通信。 <a name="PSENQ"></a>

### **HTTPS的缺点**

- HTTPS协议多次握手，导致页面的加载时间延长近50%；
- HTTPS连接缓存不如HTTP高效，会增加数据开销和功耗；
- 申请SSL证书需要钱，功能越强大的证书费用越高。
- SSL涉及到的安全算法会消耗 CPU 资源，对服务器资源消耗较大。

<a name="SZ6Qt"></a>

## 对称加密

<a name="kOKLq"></a>

### 概念

非对称加密需要两个密钥：公钥 (publickey) 和私钥 (privatekey)。公钥和私钥是一对，如果用公钥对数据加密，那么只能用对应的私钥解密。如果用私钥对数据加密，只能用对应的公钥进行解密。因为加密和解密用的是不同的密钥，所以称为非对称加密。
非对称加密算法的保密性好，它消除了最终用户交换密钥的需要。但是加解密速度要远远慢于对称加密，在某些极端情况下，甚至能比对称加密慢上1000倍。 <a name="Qc8MH"></a>

### 特点

算法强度复杂、安全性依赖于算法与密钥但是由于其算法复杂，而使得加密解密速度没有对称加密解密的速度快。对称密码体制中只有一种密钥，并且是非公开的，如果要解密就得让对方知道密钥。所以保证其安全性就是保证密钥的安全，而非对称密钥体制有两种密钥，其中一个是公开的，这样就可以不需要像对称密码那样传输对方的密钥了。这样安全性就大了很多。

<a name="qeSwu"></a>

### RSA算法

<a name="KJwTW"></a>

### DSA算法

<a name="MiQkz"></a>

### ECC算法

<a name="vGfZb"></a>

### DH算法

<a name="qCnj0"></a>

## 非对称加密
