---
title: bind
url: https://www.yuque.com/endday/blog/pumr4h
---

[new](new.md)

[面试官问：能否模拟实现JS的bind方法](https://juejin.im/post/5bec4183f265da616b1044d7)

作者：若川
链接：<https://juejin.im/post/5bec4183f265da616b1044d7>
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

```javascript
var obj = {};
console.log(obj);
// bind本身是个方法，返回值也是一个方法
console.log(typeof Function.prototype.bind); // function
console.log(typeof Function.prototype.bind());  // function
// bind方法的name为bind，返回的方法的name为bound
console.log(Function.prototype.bind.name);  // bind
console.log(Function.prototype.bind().name);  // bound
```

1. bind本身是个方法，返回值也是一个方法
2. bind方法的name为bind，返回的方法的name为bound

```javascript
var obj = {
    name: 'baby',
};
function original(a, b){
    console.log(this.name);
    console.log([a, b]);
    return false;
}
// 在使用bind方法传入的除第一个参数以外的参数
// 和执行bound函数传入的参数会拼接起来
var bound = original.bind(obj, 1);
// 在bind时传入了一个参数1，实际bound函数执行传入2
// 拼接出的实参其实是（1，2）
var boundResult = bound(2); // 'baby', [1, 2]
var bound = original.bind(obj);
// 这里bound执行时的实参就是（2）
var boundResult = bound(2); // 'baby', [2, undefined]

console.log(boundResult); // false
console.log(original.bind.name); // 'bind'
console.log(original.bind.length); // 1 bind的形参个数为1，就是this指向
console.log(original.bind().length); // 2 返回original函数的形参个数
// bind()返回函数的name为bound + 空格 + 调用bind的函数名
console.log(bound.name); // 'bound original'
console.log((function(){}).bind().name); // 'bound '
console.log((function(){}).bind().length); // 0
```

1. 在使用bind方法传入的除第一个参数以外的参数，和执行bound函数传入的参数会拼接起来
2. bind返回的函数的形参个数和原函数保持一致

```typescript
Function.prototype.bindFn = function bind(thisArg){
    if(typeof this !== 'function'){
        throw new TypeError(this + ' must be a function');
    }
    // 存储调用bind的函数本身
    var self = this;
    // 去除thisArg的其他参数 转成数组
    var args = [].slice.call(arguments, 1);
    var bound = function(){
        // bind返回的函数 的参数转成数组
        var boundArgs = [].slice.call(arguments);
        var finalArgs = args.concat(boundArgs);
        // 判断bound函数是否进行 new 调用，其实this instanceof bound判断也不是很准确。
      	// es6 new.target就是解决这一问题的。
        if(this instanceof bound){
          	// 开始模拟new的调用
            // 这里是实现上文描述的 new 的第 1, 2, 4 步
            // self可能是ES6的箭头函数，没有prototype，所以就没必要再指向做prototype操作。
            if(self.prototype){
                // ES5 提供的方案 Object.create()
        				// bound.prototype = Object.create(self.prototype);
        				// 这是个ployfill
                function Empty(){} // 1.创建一个全新的对象
                Empty.prototype = self.prototype; // 2.并且执行[[Prototype]]链接
                bound.prototype = new Empty(); // 4.通过`new`创建的每个对象将最终被`[[Prototype]]`链接到这个函数的`prototype`对象上。
            }
            // 3.生成的新对象会绑定到函数调用的`this`。
            var result = self.apply(this, finalArgs);
            // 这里是实现上文描述的 new 的第 5 步
            // 5.如果函数没有返回对象类型`Object`(包含`Functoin`, `Array`, `Date`, `RegExg`, `Error`)，
            // 那么`new`表达式中的函数调用会自动返回这个新的对象。
            var isObject = typeof result === 'object' && result !== null;
            var isFunction = typeof result === 'function';
            if(isObject || isFunction){
                return result;
            }
            return this;
        }
        else{
            // apply修改this指向，把两个函数的参数合并传给self函数，并执行self函数，返回执行结果
            return self.apply(thisArg, finalArgs);
        }
    };
    return bound;
}
```
