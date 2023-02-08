---
title: observer模块浅析
url: https://www.yuque.com/endday/blog/uv30rt
---

> 当对象间存在一对多关系时，则使用观察者模式（Observer Pattern）。
> 比如，当一个对象被修改时，则会自动通知依赖它的对象。观察者模式属于行为型模式。

<a name="UfU1u"></a>

# 模块文件

- **index.js **
  - 主要用于将对象转换为响应式对象（Object.defineProperty） 
  - Observer、defineReactive
- **watcher.js**
  - 观察者
- **dep.js**
  - 依赖，用来收集观察者
- **array.js**
  - 劫持数组方法，当修改数组时会触发更新
- **traverse.js**
  - 递归对象，进行对象的深度监听，进行依赖收集，用于watch的deep选项为true时，还有createElement中
- **scheduler.js**
  - 这是个负责调度的方法，更合理的执行watcher更新 

<a name="U79KI"></a>

# 流程顺序

<a name="MQMIW"></a>

#### new Vue()

<a name="PoRU0"></a>

#### initRender(vm)

- Vue.prototype.\_render <a name="RAQ6u"></a>

#### initState(vm)

- new Observe
  - this.dep = new Dep()
  - 数组：劫持数组内置方法
  - 对象：defineReactive方法：修改defineProperty的get和set
    - get: dep.depend() 收集watcher
    - set: dep.notify() 通知watcher更新 <a name="TBTsd"></a>

#### vm.$mount()

根组件挂载

- new Watcher(vm, expOrFn)
  - **construction**
    - 将expOrFn转换为function。expOrFn有可能是表达式，也可能是function
      - this.getter = expOrFn;
      - this.getter = parsePath(expOrFn);
    - Watch.prototype.get()
      - pushTarget(this)
        - targetStack.push(target);
        - Dep.target = target;
      - let value = this.getter.call(vm, vm);  触发Object.defineProperty的get方法
        - dep.depend(); 收集watcher
- 3种watcher
  - 当是渲染watcher时，expOrFn是updateComponent，即重新渲染执行render（\_update）
  - 当是计算watcher时，expOrFn是计算属性的计算方法
  - 当是侦听器watcher时，expOrFn是watch属性的名字，this.cb就是watch的handler属性 <a name="2J9VW"></a>

#### vm.\_update(vm.\_render(), hydrating)

- 这里的expOrFn传的是updateComponent，包着vm.\_update
- render 是返回vnode的函数，即产生虚拟dom <a name="jXIn3"></a>

#### vm.**patch**

- createElm()
- createComponent(vnode)
- vnode.data.hook.init() <a name="O8ho3"></a>

#### child.$mount（vm.$mount）

<a name="hlw8t"></a>

# 图例

![image.png](..\\..\\..\assets\uv30rt\1578132143879-bfefe2e2-c991-4791-a018-5722c5615f94.png)
![image.png](..\\..\\..\assets\uv30rt\1578310137669-39f9b0a8-ed74-461b-9a7f-5933361be659.png)

```javascript
export default {
  name: 'app',
  components: {
    HelloWorld,
    HelloWorld1
  },
  mounted () {
    console.log(this)
    console.log(this.$data)
    // 第一个读取组件内data触发依赖收集的，就是组件的视图template中的渲染
    // 一个组件内只存在一个render函数，所以如果在template中使用到的变量，都会收集到render-watcher
    console.log(this.test.__ob__.dep.subs[0] === this.a.__ob__.dep.subs[0])
    // 这个render-watcher 是同一个wather对象
  },
  computed: {
    testComputed () {
      return this.test.msg + 1
    },
    testComputed2 () {
      return this.test.msg + 2
    }
  },
  data () {
    return {
      test: {
        msg: 'Welcome to Your Vue.js App'
      },
      a: [
        1,
        2,
        3
      ]
    }
  }
}
```

![](..\\..\\..\assets\uv30rt\1599125819409-e2f502b0-297d-444c-a85b-3fec70548e08.png)![reactive.png](..\\..\\..\assets\uv30rt\1598973977499-b3a971a0-e15f-45c6-8d70-e1baefc5e4c6.png)
[vue observer模块浅析.ppt](https://www.yuque.com/attachments/yuque/0/2020/ppt/122551/1599115766033-f6572d37-ebb3-4e00-bf39-9be6afdc7017.ppt)

重点关注

1. 响应式数据和依赖收集的过程
2. 三种watcher的异同
3. scheuler调度方法对watcher队列的执行

疑惑点
observer/index.js
// [#7981: for accessor properties without setter](https://github.com/vuejs/vue/issues/9203)
![image.png](..\\..\\..\assets\uv30rt\1599126390683-fc708a19-75dd-4e94-b6d4-96e166793a03.png)
![image.png](..\\..\\..\assets\uv30rt\1599126370251-cf024f5a-0be9-4c47-920e-060b35bff085.png)
