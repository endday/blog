---
title: javascript跨域通信
url: https://www.yuque.com/endday/blog/moogva
---

同源：两个文档同源需满足

- 协议相同
- 域名相同
- 端口相同

跨域通信：js进行DOM操作、通信时如果目标与当前窗口不满足同源条件，浏览器为了安全会阻止跨域操作。跨域通信通常有以下方法

- 如果是log之类的简单单项通信，新建,<script>,,<iframe>元素，通过src，href属性设置为目标url。实现跨域请求
- 如果请求json数据，使用<script>进行jsonp请求
- 现代浏览器中多窗口通信使用HTML5规范的targetWindow.postMessage(data, origin);其中data是需要发送的对象，origin是目标窗口的origin。window.addEventListener('message', handler, false);handler的event.data是postMessage发送来的数据，event.origin是发送窗口的origin，event.source是发送消息的窗口引用
- 内部服务器代理请求跨域url，然后返回数据
- 跨域请求数据，现代浏览器可使用HTML5规范的CORS功能，只要目标服务器返回HTTP头部**Access-Control-Allow-Origin: ***即可像普通ajax一样访问跨域资源
