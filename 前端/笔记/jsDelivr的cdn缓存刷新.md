---
title: jsDelivr的cdn缓存刷新
url: https://www.yuque.com/endday/blog/xs8px1
---

对于 jsDelivr，缓存刷新的方式也很简单，只需将想刷新的链接的开头的
`https://cdn.jsdelivr.net/...`
替换成
`https://purge.jsdelivr.net/...`
即可实时刷新。刷新成功后，浏览器会返回缓存刷新成功的信息，如下：
![image.png](..\\..\assets\xs8px1\1599146358977-35f10732-9cf0-47f0-86f3-889417c445c3.png)
