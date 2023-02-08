---
title: 使用Blob对象来下载静态资源
url: https://www.yuque.com/endday/blog/at7e6o
---

```javascript
const getBlob = (url) => {
  return new Promise((resolve) => {
    const xhr = new XMLHttpRequest()
    xhr.open('GET', url, true)
    xhr.responseType = 'blob'
    xhr.onload = () => {
      if (xhr.status === 200) {
        resolve(xhr.response)
      }
    }
    xhr.send()
  })
}

const saveAs = (blob, filename) => {
  // for IE
  if (window.navigator && window.navigator.msSaveOrOpenBlob) {
    window.navigator.msSaveBlob(blob, filename)
  } else {
    const link = document.createElement('a')
    link.href = window.URL.createObjectURL(blob) // 创建对象url
    link.download = filename
    link.style.display = 'none'
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
    window.URL.revokeObjectURL(link.href) // 静态方法用来释放对象url
  }
}

export default function (url, filename = '') {
  getBlob(url).then((blob) => {
    saveAs(blob, filename)
  })
}
```
