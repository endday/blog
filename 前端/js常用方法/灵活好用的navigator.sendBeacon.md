---
title: 灵活好用的navigator.sendBeacon
url: https://www.yuque.com/endday/blog/cliv38
---

> 这个方法主要用于满足 统计和诊断代码 的需要，这些代码通常尝试在卸载(unload)文档之前向web服务器发送数据。过早的发送数据可能导致错过收集数据的机会。然而， 对于开发者来说保证在文档卸载期间发送数据一直是一个困难。因为用户代理通常会忽略在卸载事件处理器中产生的异步 XMLHttpRequest 。

<a name="G2tEm"></a>

#### DOMString

如果数据类型是 `string`，则可以直接上报，此时该请求会自动设置请求头的 `Content-Type` 为 `text/plain`。


    const reportData = (url, data) => {
      navigator.sendBeacon(url, data);
    };

<a name="k7NHi"></a>

#### Blob

如果用 `Blob` 发送数据，这时需要我们手动设置 `Blob` 的 MIME type，一般设置为 `application/x-www-form-urlencoded`。


    const reportData = (url, data) => {
      const blob = new Blob([JSON.stringify(data), {
        type: 'application/x-www-form-urlencoded',
      }]);
      navigator.sendBeacon(url, blob);
    };

<a name="skJgs"></a>

#### Formdata

可以直接创建一个新的 `Formdata`，此时该请求会自动设置请求头的 `Content-Type` 为 `multipart/form-data`。

    const reportData = (url, data) => {
      const formData = new FormData();
      Object.keys(data).forEach((key) => {
        let value = data[key];
        if (typeof value !== 'string') {
          // formData只能append string 或 Blob
          value = JSON.stringify(value);
        }
        formData.append(key, value);
      });
      navigator.sendBeacon(url, formData);
    };

注意这里的 `JSON.stringify` 操作，服务端需要将数据进行 parse 才能得到正确的数据。
