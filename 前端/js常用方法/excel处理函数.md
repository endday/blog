---
title: excel处理函数
url: https://www.yuque.com/endday/blog/gvawf9
---

<a name="J0HVc"></a>

# excel转json

```javascript
const Excel = require('exceljs/modern.browser')
const fs = require('fs')
var workbook = new Excel.Workbook()

workbook.xlsx.readFile('./133820评论数据.xlsx')
  .then(function () {
    var worksheet = workbook.getWorksheet(1)
    let dataArray = changeRowsToDict(worksheet)
    const json = JSON.stringify(dataArray)

    fs.writeFile('excel.json', json, 'utf8', function (error) {
      if (error) {
        console.log(error)
        return false
      }
      console.log('写入成功')
    })
  })

/* 将所有的行数据转换为json */
function changeRowsToDict (worksheet) {
  let dataArray = []
  let keys = []
  worksheet.eachRow(function (row, rowNumber) {
    if (rowNumber === 1) {
      keys = row.values
    } else {
      // method1  ===============
      // let rowDict = cellValueToDict(keys, row.values);
      // dataArray.push(rowDict);
      // method2  ===============
      let rowDict = cellValueToDict2(keys, row)
      dataArray.push(rowDict)
    }
  })
  return dataArray
}

/* keys: {id,name,phone}, rowValue：每一行的值数组, 执行次数3次 */
function cellValueToDict (keys, rowValue) {
  let rowDict = {}
  keys.forEach((value, index) => {
    rowDict[value] = rowValue[index]
  })
  return rowDict
}

/* keys: {id,name,phone}, rowValue：每一行的值数组， 执行次数3次 */
function cellValueToDict2 (keys, row) {
  let data = {}
  row.eachCell(function (cell, colNumber) {
    var value = cell.value
    if (typeof value === 'object') value = value.text
    data[keys[colNumber]] = value
  })
  return data
}

```

<a name="n8lOI"></a>

# json文件处理

```javascript
const fs = require('fs')
const filePath = 'excel.json'

fs.readFile(filePath, function (err, data) {
  if (err) {
    return console.error(err)
  }
  // console.log('异步读取: ' + data.toString())
  const json = JSON.parse(data).slice(0, 50)
  const list = JSON.stringify(json)
  fs.writeFile(filePath, list, 'utf8', function (error) {
    if (error) {
      console.log(error)
      return false
    }
    console.log('写入成功')
  })
})
```
