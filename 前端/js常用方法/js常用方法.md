---
title: js常用方法
url: https://www.yuque.com/endday/blog/gm5ioz
---

<a name="510a44e6"></a>

### 深拷贝

```javascript
var cloneObj = function(obj){
    var str, newobj = obj.constructor === Array ? [] : {};
    if(typeof obj !== 'object'){
        return;
    } else if(window.JSON){
        str = JSON.stringify(obj), //系列化对象
        newobj = JSON.parse(str); //还原
    } else {
        for(var i in obj){
            newobj[i] = typeof obj[i] === 'object' ? 
            cloneObj(obj[i]) : obj[i]; 
        }
    }
    return newobj;
};
```

<a name="e3caa55e"></a>

### 数组拍平

<a name="6a4f83e5"></a>

#### 拍平数组（一层）

```javascript
const flatten = [1, [2, [3, [4]], 5]].reduce( (a, b) => a.concat(b), [])
// => [1, 2, [3, [4]], 5]
```

<a name="64848fb8"></a>

#### 拍平数组（完全）

```javascript
const flattenDeep = (arr) => Array.isArray(arr)
  ? arr.reduce( (a, b) => [...flattenDeep(a), ...flattenDeep(b)] , [])
  : [arr]

flattenDeep([1, [[2], [3, [4]], 5]])
// => [1, 2, 3, 4, 5]
```

<a name="a26c2df0"></a>

### node获取当前文件目录名称

```javascript
path.resolve(__dirname).split(path.sep).pop()
```

<a name="dsXPP"></a>

### 获取当前月份的第一天和最后一天

```javascript
function timeFormat (date) {
    if (!date || typeof (date) === 'string') {
        throw Error('参数异常，请检查...')
    }
    var y = date.getFullYear() //年
    var m = date.getMonth() + 1 //月
    var d = date.getDate() //日
    return y + '-' + m + '-' + d
}

function getMonthFirstDay () {
    var date = new Date()
    date.setDate(1)
    return timeFormat(date)
}

function getMonthLastDay () {
    var curDate = new Date()
    var curMonth = curDate.getMonth()
    curDate.setMonth(curMonth + 1)
    curDate.setDate(0)
    return timeFormat(curDate)
}
```

<a name="j0ad7"></a>

### JS判断字符串长度（英文占1个字符，中文汉字占2个字符）

```javascript
String.prototype.gblen = function() {    
    var len = 0;    
    for (var i=0; i<this.length; i++) {    
        if (this.charCodeAt(i)>127 || this.charCodeAt(i)==94) {    
             len += 2;    
         } else {    
             len ++;    
         }    
     }    
    return len;    
}
```

```javascript
function strlen(str){  
    var len = 0;  
    for (var i=0; i<str.length; i++) {   
     var c = str.charCodeAt(i);   
    //单字节加1   
     if ((c >= 0x0001 && c <= 0x007e) || (0xff60<=c && c<=0xff9f)) {   
       len++;   
     }   
     else {   
      len+=2;   
     }   
    }   
    return len;  
}
```

```javascript
var jmz = {};  
jmz.GetLength = function(str) {  
    ///<summary>获得字符串实际长度，中文2，英文1</summary>  
    ///<param name="str">要获得长度的字符串</param>  
    var realLength = 0, len = str.length, charCode = -1;  
    for (var i = 0; i < len; i++) {  
        charCode = str.charCodeAt(i);  
        if (charCode >= 0 && charCode <= 128) realLength += 1;  
        else realLength += 2;  
    }  
    return realLength;  
};
```

```javascript
var l = str.length;   
var blen = 0;   
for(i=0; i<l; i++) {   
	if ((str.charCodeAt(i) & 0xff00) != 0) {   
		blen ++;   
	}   
	blen ++;   
}
```

```javascript
getBLen = function(str) {  
    if (str == null) return 0;  
    if (typeof str != "string"){  
        str += "";  
    }  
    return str.replace(/[^\x00-\xff]/g,"01").length;  
}
```

<a name="JNvBa"></a>

### 原生ajax步骤

```javascript
function ajax(url, success, fail) {
  // 1. 创建连接
  const xhr = new XMLHttpRequest()
  // 2. 连接服务器
  // 规定请求的类型、URL 以及是否异步处理请求。
  xhr.open('get', url, true)
  // 发送信息至服务器时内容编码类型
  xhr.setRequestHeader('Content-type', 'application/x-www-form-urlencoded')
  // 3. 发送请求
  xhr.send(null)
  // 4. 接受服务器响应数据
  xhr.onreadystatechange = function() {
    if (xhr.readyState === 4) {
      if ((xhr.status >= 200 && xhr.status < 300) || xhr.status === 304) {
        success(xhr.responseText)
      } else { // fail
        fail && fail(xhr.status)
      }
    }
  }
}
```

<a name="vCdgK"></a>

### *驼峰式和下划线的转换*

<a name="7Pmi5"></a>

#### \_驼峰转下划线

```javascript
const toLowerLine = function (str) {
    return str.replace(/\B([A-Z])/g, '_$1').toLowerCase()
}
```

<a name="4zEuQ"></a>

#### 下划线转驼峰

```javascript
const toLowerLine = function (str) {
    return str.replace(/\B([A-Z])/g, '_$1').toLowerCase()
}
```

<a name="Vp1cJ"></a>

#### 整个对象进行转换

```javascript
const listFormatCamel = function (obj) {
    if (Array.isArray(obj)) {
        obj.forEach((v, i) => {
            listFormatCamel(v)
        })
    } else if (obj instanceof Object) {
        Object.keys(obj).forEach(key => {
            const newKey = toCamel(key)
            if (newKey !== key) {
                obj[newKey] = obj[key]
                delete obj[key]
            }
            listFormatCamel(obj[newKey])
        })
    }
}
```

<a name="XfKQc"></a>

### 捕获未catch的错误

```javascript
window.addEventListener("unhandledrejection", function(promiseRejectionEvent) { 
    // handle error here, for example log   
});
```

<a name="Jxjor"></a>

### rem布局的新招式，用vw替代resize的监听方法

```javascript
// rem 单位换算：定为 75px 只是方便运算，750px-75px、640-64px、1080px-108px，如此类推
$vw_fontsize: 75; // iPhone 6尺寸的根元素大小基准值
@function rem($px) {
     @return ($px / $vw_fontsize ) * 1rem;
}

// 根元素大小使用 vw 单位
$vw_design: 750;
html {
    font-size: ($vw_fontsize / ($vw_design / 2)) * 100vw; 
    // 同时，通过Media Queries 限制根元素最大最小值
    @media screen and (max-width: 320px) {
        font-size: 64px;
    }
    @media screen and (min-width: 540px) {
        font-size: 108px;
    }
}

// body 也增加最大最小宽度限制，避免默认100%宽度的 block 元素跟随 body 而过大过小
body {
    max-width: 540px;
    min-width: 320px;
}
```

<a name="1QIjl"></a>

### 函数必须传参的写法

```javascript
const isRequired = () => { throw new Error('param is required'); };

const hello = (name = isRequired()) => { console.log(`hello ${name}`) };

// This will throw an error because no name is provided
hello();

// This will also throw an error
hello(undefined);

// These are good!
hello(null);
hello('David');
```
