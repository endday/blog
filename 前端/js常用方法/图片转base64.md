---
title: 图片转base64
url: https://www.yuque.com/endday/blog/nbe3ld
---

```javascript
getBase64Image (src) {
  const getBase64Image = function (img, type) {
    let canvas = document.createElement('canvas')
    canvas.width = img.width
    canvas.height = img.height
    let ctx = canvas.getContext('2d')
    ctx.drawImage(img, 0, 0, img.width, img.height)
    return canvas.toDataURL(`image/${type}`)
  }
  const getType = function (upFileName) {
    let index1 = upFileName.lastIndexOf('.')
    let index2 = upFileName.length
    let type = upFileName.substring(index1 + 1, index2)
    if (type === 'jpg') {
      return 'jpeg'
    }
    return type
  }
  return new Promise(resolve => {
    let img = document.createElement('img')
    img.crossOrigin = 'Anonymous'
    img.onload = function () {
      let data = getBase64Image(img, getType(src))
      resolve(data)
    }
    img.src = src
  })
}
```
