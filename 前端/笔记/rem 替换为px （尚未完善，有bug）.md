---
title: rem 替换为px （尚未完善，有bug）
url: https://www.yuque.com/endday/blog/zbc97h
---

```javascript
const fs = require('fs')
const filePath = '../component/twoEggs/index.vue'

function replace (str) {
    var reg = /.?(\d)+(\.\d+)?(rem)/gi
    var arr = str.match(reg)
    for (let i = 0, len = arr.length; i < len; i++) {
        const item = arr[i]
        const num = Number(item.slice(0, item.length - 3))
        let replaceNum = num * 20000 / 100 + 'px'
        if (arr[i].length <= replaceNum.length) {
            replaceNum = ` ${replaceNum} `
        }
        str = str.replace(arr[i], replaceNum)
    }
    return str
}

fs.readFile(filePath, 'utf8', (err, data) => {
    if (err) throw err
    const formatData = replace(data)
    fs.writeFile(filePath, formatData, 'utf8', (err) => {
        if (err) throw err
        console.log('文件已保存')
    })
})
```
