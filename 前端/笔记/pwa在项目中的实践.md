---
title: pwa在项目中的实践
url: https://www.yuque.com/endday/blog/pwa
---

参考文档

- [Lavas](https://lavas.baidu.com/pwa)
- [Workbox webpack Plugins](https://developers.google.cn/web/tools/workbox/modules/workbox-webpack-plugin#configuration)
- [谨慎处理 Service Worker 的更新](https://zhuanlan.zhihu.com/p/51118741)

[demo地址](https://endday.github.io/pwa-vue-cli-demo/#/)
[项目地址](https://github.com/endday/pwa-vue-cli-demo)

<a name="tLuhP"></a>

# 简介

**PWA和Service Worker的关系**

> PWA (Progressive Web Apps) 不是一项技术，也不是一个框架，我们可以把她理解为一种模式，一种通过应用一些技术将 Web App 在安全、性能和体验等方面带来渐进式的提升的一种 Web App的模式。
> 对于 webview 来说，Service Worker 是一个独立于js主线程的一种 Web Worker 线程， 一个独立于主线程的 Context，但是面向开发者来说 Service Worker 的形态其实就是一个需要开发者自己维护的文件，我们假设这个文件叫做 sw.js。
> 通过 service worker 我们可以代理 webview 的请求相当于是一个正向代理的线程，fiddler也是干这些事情），在特定路径注册 service worker 后，可以拦截并处理该路径下所有的网络请求，进而实现页面资源的可编程式缓存，在弱网和无网情况下带来流畅的产品体验，所以 service worker 可以看做是实现pwa模式的一项技术实现。

**service worker简介**

- service worker 是一种JS工作线程，无法直接访问DOM, 该线程通过postMessage接口消息形式来与其控制的页面进行通信;

![](..\\..\assets\pwa\1564133901160-7942ad1a-e8fe-42ed-b0e9-544b116f9104.jpeg) <a name="mGM8I"></a>

### 生命周期

一个service worker在启动前经历了三步：

- 注册（Registration)
- 安装（Installation）
- 激活（Activation）
- 更新（updated）

![](..\\..\assets\pwa\8ba1d2dce1faab92fe02505f1c4a6e08.svg)

> service worker的生命周期完全独立于网页，要为网站安装服务工作线程，我们需要在页面业务js代码中**注册**，浏览器从指定路径下载并解析服务工作线程脚本进而浏览器将会在后台启动**安装**步骤，在安装过程中，我们通常会缓存静态资源，如果所有文件都成功缓存，那么服务工程线程就安装完毕，如果任何文件下载失败或缓存失败，那么安装步骤将会失败，当然也不会被激活。安装后就进入**激活**步骤，这里是管理旧缓存的绝佳机会（后面代码示例中将会介绍原因），激活后service worker将开始对其作用域内的所有页面实施控制。这里需要注意的是，首次注册 service worker 线程的页面需要再次加载才会受其控制。在成功安装完成并处于激活状态之前，服务工程线程不会收到fetch和push事件;

<a name="6PIG0"></a>

#### 注册

注册阶段是通知浏览器service worker的存在，并且在后台开始安装。 <a name="m3WiK"></a>

#### 安装( installing )

这个状态发生在 Service Worker 注册之后，表示开始安装，触发 install 事件回调指定一些静态资源进行离线缓存。 <a name="GpA0S"></a>

#### 安装后( installed )

Service Worker 已经完成了安装，并且等待其他的 Service Worker 线程被关闭。 <a name="laXhz"></a>

#### 激活( activating )

在这个状态下没有被其他的 Service Worker 控制的客户端，允许当前的 worker 完成安装，并且清除了其他的 worker 以及关联缓存的旧缓存资源，等待新的 Service Worker 线程被激活。 <a name="aKZXe"></a>

#### 激活后( activated )

在这个状态会处理 activate 事件回调 (提供了更新缓存策略的机会)。并可以处理功能性的事件 fetch (请求)、sync (后台同步)、push (推送)。 <a name="EBxGd"></a>

#### 废弃状态 ( redundant )

这里特别说明一下，这个状态表示一个 Service Worker 的生命周期结束。
进入废弃 (redundant) 状态的原因可能为这几种：

- 安装 (install) 失败

- 激活 (activating) 失败

- 新版本的 Service Worker 替换了它并成为激活状态

<a name="As7Uq"></a>

# 配置

用到的依赖

- vue-cli3
- @vue/cli-plugin-pwa
- workbox
- [sw-register-webpack-plugin](https://github.com/lavas-project/sw-register-webpack-plugin) <a name="utLFm"></a>

### service-worker注册

```javascript
let path = '/sw-test/sw.js'
let scope = '/sw-test/'
navigator.serviceWorker.register(path, { scope }).then(function(reg) {
    // registration worked
    console.log('Registration succeeded. Scope is ' + reg.scope);
  }).catch(function(error) {
    // registration failed
    console.log('Registration failed with ' + error);
  });
```

1. **path**通常理解为**路径**，就是service-worker存放的位置，也可以理解为请求service-worker的url地址。可以通过后端请求将service-worker从其他目录展示到你需要的Url地址下。
2. **scope** 理解为**作用域**，意义是在该作用域以及其下级目录下发起的fetch请求都受当前的service-worker控制，在作用域以外地址下发起的请求，sw是无法进行代理的。  
3. 在不填写scope的情况下，默认的scope就是path的父级目录，上图的path是/sw-test/sw.js，默认scope就是/sw-test/。 
4. 配置scope只能在默认作用域，也就是path的范围内再自定义，相当于只能缩小作用域，不能扩大作用域的范围。假如默认scope为/a/b/，可以通过传入{scope: '/a/b/c/'}来指定自己的scope，自定义为/d/e/就不行。 <a name="rwOqd"></a>

### service-worker更新和缓存 

> - 下列情况会触发一次更新（浏览器检查service worker的更新）
>   - 访问作用域下的页面
>   - 当`push`和`sync`等功能性事件发生时，除非在之前的24小时之内做过更新检查
>   - 在service worker的URL改变后调用`register()`
> - service-worker.js也会受http的缓存策略控制
> - 如果新的worker未被成功下载，或者解析错误，或者在运行时出错，或者在安装阶段不成功，新的worker会被丢弃，旧的会被保留
> - 一旦新的worker被成功安装，更新的worker会进入等待状态，新的worker会等待旧的worker下线才会激活，新的worker和旧的会并存
> - `self.skipWaiting()`会强制跳过等待状态，直接让新的worker在安装后进入激活状态，这样可能会有缓存问题
> - 浏览器会 diff 当前打开页面的 service-worker.js，并判断是否更新，如果 diff 结果为更新，则重新安装最新的 service-wroker.js，并且全量更新缓存
> - 任何静态资源包括 service-worker.js 都会被 HTTP 缓存
> - 服务器对某个资源进行 `no-cache` 设置可以避免 HTTP 缓存

针对上述的情况，service-worker的更新就是必须解决的问题。
下面分两种方法

1. 在服务器端配置service-worker的header，Cache-control: no-cache，使其不被缓存
2. 前端进行service-worker的版本控制，每次注册都添加版本号进行改写

- 下面是一种简单粗暴的解决方法，缺点就是每次会重新请求service-worker

```javascript
// sw-register.js
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/service-worker.js' + Date.now()).then(function (reg) {
    
  })
}
```

- 利用 `sw-register-webpack-plugin`  插件，可以自动生成版本号，[github地址](https://github.com/lavas-project/sw-register-webpack-plugin)

```javascript
// npm install sw-register-webpack-plugin --save-dev

const SwRegisterWebpackPlugin = require('sw-register-webpack-plugin')

webpack({
    // ...
    plugins: [
        new SwRegisterWebpackPlugin(/* options */);
    ]
    // ...
});
```

- 折中的方法，用webpack.DefinePlugin插件，将版本号替

```javascript
// webpack.config.jg
const webpack = require('webpack')

function getVersion () {
  var d = new Date()
  return '' + d.getFullYear() + d.getMonth() + 1 + d.getDate() + d.getHours() + d.getMinutes() + d.getSeconds()
}

webpack({
    // ...
  plugins: [
    new webpack.DefinePlugin({
      __SW_VERSION__: getVersion()
    })
  ]
  // ...
});
```

```javascript
// sw-register.js
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/service-worker.js?version=' + __SW_VERSION__)
    .then(function (reg) {
      
    })
    .catch(function (e) {
      
    })
}
```

<a name="JnqMR"></a>

### serivice-worker激活

1 skipWaiting跳过等待阶段
2 页面提示
3 添加加载动画，等待sw下载

> 由于浏览器的内部实现原理，当页面切换或者自身刷新时，浏览器是等到新的页面完成渲染之后再销毁旧的页面。这表示新旧两个页面中间有共同存在的交叉时间，因此简单的切换页面或者刷新是不能使得 service worker 进行更新的。

既然service-worker的激活无法通过刷新解决，那么还有个`skipWaiting`可以用。

但是最好不要直接`skipWaiting`(跳过等待阶段)， 推荐的做法应该是在浏览器发现更新后，给用户弹出提示。然后用户点击重新加载时，一方面刷新页面 (`location.reload()`)，一方面让新的 SW 接管页面 (`skipWaiting`)。

具体的流程：

- 在注册service-worker时就监听sw的更新状况
- 如果有更新，并且安装完成后，就发送自定义事件sw.update
- 自定义事件被触发，显示更新按钮
- 用户点击更新按钮触发更新

![](..\\..\assets\pwa\367db81b08ee7b493b7515bfe9739466.svg)

```javascript
function emitUpdate () {
  var event = document.createEvent('Event')
  event.initEvent('sw.update', true, true)
  window.dispatchEvent(event)
}

if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/zhangyi/sw')
    .then(function (reg) {
      if (reg.waiting) {
        emitUpdate()
        return
      }
      reg.onupdatefound = function () {
        var installingWorker = reg.installing
        installingWorker.onstatechange = function () {
          switch (installingWorker.state) {
            case 'installed':
              if (navigator.serviceWorker.controller) {
                // 自定义的更新事件
                emitUpdate()
              }
              break
          }
        }
      }
    })
    .catch(function (e) {
      console.error('Error during service worker registration:', e)
    })
}
```

```javascript
let refreshing = false

export default {
  name: 'SWUpdatePopup',
  data () {
    return {
      showSwUpdate: false
    }
  },
  mounted () {
    this.addListener()
  },
  methods: {
    addListener () {
      window.addEventListener('sw.update', this.handleUpdate)
      this.$once('hook:beforeDestroy', function () {
        window.removeEventListener('sw.update', this.handleUpdate)
      })
    },
    handleUpdate () {
      this.showSwUpdate = true
    },
    handleSkipWaiting () {
      navigator.serviceWorker.getRegistration()
        .then(reg => this.skipWaiting(reg))
        .then(() => {
          window.location.reload(true)
        })
    },
    handleSWChange () {
      if (refreshing) {
        return
      }
      refreshing = true
      window.location.reload()
    },
    skipWaiting (registration) {
      const worker = registration.waiting
      if (!worker) {
        return Promise.resolve()
      }
      return new Promise((resolve, reject) => {
        const channel = new MessageChannel()
        channel.port1.onmessage = (event) => {
          if (event.data.error) {
            reject(event.data.error)
          } else {
            resolve(event.data)
          }
        }
        worker.postMessage({ type: 'skip-waiting' }, [channel.port2])
      })
    },
    handleRefresh () {
      window.location.reload(true)
    }
  }
}
```

配置主要基于 vue-cli 的 [pwa 插件](https://github.com/vuejs/vue-docs-zh-cn/blob/master/vue-cli-plugin-pwa/README.md)和 [workbox-webpack-plugin](https://link.juejin.im/?target=https%3A%2F%2Fdevelopers.google.com%2Fweb%2Ftools%2Fworkbox%2Fmodules%2Fworkbox-webpack-plugin)

workbox-webpack-plugin主要提供两种模式：

> **GenerateSW **模式根据配置生成sw文件，适用场景：
>
> - 简单的运行时配置需求
> - 不涉及Web Push
>
> **InjectManifest **模式通过既有sw文件再加工，适用场景；
>
> - 涉及Web Push
> - 更复杂的自定义配置

这里使用的GenerateSW模式

```javascript
// vue.config.js
const { InjectManifest } = require('workbox-webpack-plugin')

module.exports = {
	configureWebpack: config => {
    config.plugins.push(
      new InjectManifest({
        swSrc: './src/service-worker.js',
        importsDirectory: 'js',
        importWorkboxFrom: 'disabled', // 不使用谷歌workerbox的cdn
        exclude: [/\.map$/, /^manifest.*\.js$/, /\.html$/]
      })
    )
  }
}
```

<a name="cbZjk"></a>

### serivice-worker卸载

当service-worker新版本的更新出现问题，那么就要考虑如何保证用户看到的版本是最新的
我选择的策略是卸载当前的sw，用线上的文件，并且不再安装当前错误版本的。

```javascript
// sw-register.js
const version = Number(__SW_VERSION__)
const project = __PROJECT_NAME__

function emitUpdate () {
  var event = document.createEvent('Event')
  event.initEvent('sw.update', true, true)
  window.dispatchEvent(event)
}

function emitUnregister () {
  var event = document.createEvent('Event')
  event.initEvent('sw.unregister', true, true)
  window.dispatchEvent(event)
}

function unregister () {
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.getRegistration()
      .then(function (registration) {
        if (registration) {
          registration.unregister().then(function () {
            emitUnregister()
          })
        }
      })
  }
}

const failSwName = 'fail:' + project + '-sw-version'

function getFailVersion () {
  const version = window.localStorage.getItem(failSwName)
  if (version) {
    return Number(version)
  }
  return ''
}

function setFailVersion () {
  window.localStorage.setItem(failSwName, version)
}

if (getFailVersion() !== version && 'serviceWorker' in navigator) {
  // 如果是新的版本，那就尝试注册安装
  navigator.serviceWorker.register(`/${project}/service-worker.js?version=${version}`) // eslint-disable-line
    .then(function (reg) {
      if (reg.waiting) {
        emitUpdate()
        return
      }
      reg.onupdatefound = function () {
        var installingWorker = reg.installing
        installingWorker.onstatechange = function () {
          if (installingWorker.state === 'installed') {
            if (navigator.serviceWorker.controller) {
              emitUpdate()
            }
          }
        }
      }
    })
    .catch(function (e) {
      console.error('Error during service worker registration:', e)
      // 注册失败后，在session中写入失败的版本，并直接卸载
      setFailVersion()
      unregister()
    })
} else {
  // 直接卸载
  unregister()
}
```

```javascript
// service-worker.js
//...
self.addEventListener('message', event => {
  const replyPort = event.ports[0]
  const message = event.data
  if (replyPort && message && message.type === 'skip-waiting') {
    event.waitUntil(
      self.skipWaiting()
        .then(() => replyPort.postMessage({ error: null }))
        .catch(error => replyPort.postMessage({ error }))
    )
  }
})
//...
```

更新的弹窗

```html
<template>
  <div>
    <div
      class="sw-update-dialog"
      v-if="showSwUpdate"
    >
      <button @click="handleSkipWaiting">
        更新
      </button>
    </div>
    <div
      class="sw-update-dialog"
      v-if="showSwUnregister"
    >
      <button @click="handleRefresh">
        更新
      </button>
    </div>
  </div>
</template>

<script>
let refreshing = false

export default {
  name: 'SWUpdatePopup',
  data () {
    return {
      showSwUpdate: false,
      showSwUnregister: false
    }
  },
  mounted () {
    this.addListener()
  },
  methods: {
    addListener () {
      window.addEventListener('sw.update', this.handleUpdate)
      window.addEventListener('sw.unregister', this.handleUnregister)
      this.$once('hook:beforeDestroy', function () {
        window.removeEventListener('sw.update', this.handleUpdate)
        window.removeEventListener('sw.unregister', this.handleUnregister)
      })
    },
    handleUpdate () {
      this.showSwUpdate = true
    },
    handleSkipWaiting () {
      navigator.serviceWorker.getRegistration()
        .then(reg => this.skipWaiting(reg))
        .then(() => {
          window.location.reload(true)
        })
    },
    handleSWChange () {
      if (refreshing) {
        return
      }
      refreshing = true
      window.location.reload()
    },
    skipWaiting (registration) {
      const worker = registration.waiting
      if (!worker) {
        return Promise.resolve()
      }
      // 这里是参考vue-press的写法
      // 利用MessageChannel返回一个promise
      return new Promise((resolve, reject) => {
        const channel = new MessageChannel()
        channel.port1.onmessage = (event) => {
          if (event.data.error) {
            reject(event.data.error)
          } else {
            resolve(event.data)
          }
        }
        worker.postMessage({ type: 'skip-waiting' }, [channel.port2])
      })
    },
    handleUnregister () {
      this.showSwUnregister = true
    },
    handleRefresh () {
      window.location.reload(true)
    }
  }
}
</script>
```

<a name="Sc2NJ"></a>

### 缓存策略

> <a name="11abe492"></a>

#### Cache Only

> 只从缓存中读取，当缓存中没有数据时，读取失败。 <a name="ee69575d"></a>

#### NetWork Only

> 只通过网络请求进行资源请求，若请求失败，则返回失败响应。 <a name="90fe7635"></a>

#### NetWork First

> - 优先网络请求
> - 网络请求成功时，将结果写入缓存，并将结果直接返回。
> - 网络请求失败时，从缓存中读取结果，若读取结果，则返回，若未读取到，则请求失败。 <a name="d58ea2de"></a>

#### Cache First

> - 优先从缓存中读取结果
> - 若缓存中不存在结果，则进行网络请求
> - 网络请求成功时，将结果写入缓存并返回
> - 网络请求失败时，返回失败响应 <a name="8cd42dda"></a>

#### Stale While Revalidate

> - 优先查缓存，并同时发起网络请求
> - 若缓存命中且网络请求成功，返回缓存结果，并更新缓存（下次从缓存中读取的数据就是最新的了）
> - 若缓存未命中，则看网络请求是否成功，成功则更新缓存并返回结果，失败则返回失败响应。

<a name="VJZ93"></a>

#### 推荐的缓存策略

- html使用**NetWork First**当然更为稳妥，只在网络状况较差的情况发挥作用
- css，js和img等等静态资源
  - 资源名称带hash的，使用**Cache First，**优先使用缓存，有缓存，就不发起请求，节省流量
  - 不带hash，使用**Stale While Revalidate**，使用缓存，并发起请求缓存资源
- 接口使用的就要看需求了
  - 对实时性要求不高的接口，可以用**Stale While Revalidate**
  - 对实时性要求高的接口，直接就不要受sw控制，或者**NetWork First**

```javascript
// service-worker.js

// html的缓存策略
workbox.routing.registerRoute(
  new RegExp(''.*\.html'),
  workbox.strategies.networkFirst()
);

workbox.routing.registerRoute(
  new RegExp('.*\.(?:js|css)'),
  workbox.strategies.cacheFirst()
);

workbox.routing.registerRoute(
  new RegExp('https://your\.cdn\.com/'),
  workbox.strategies.staleWhileRevalidate()
);

workbox.routing.registerRoute(
  new RegExp('https://your\.img\.cdn\.com/'),
  workbox.strategies.cacheFirst({
    cacheName: 'example:img'
  })
);
```

<a name="pa3NR"></a>

# 项目示例

这是和文章内容基本一致的一个demo，大家可以测试下访问后的service-worker的情况，还有离线的访问能力
[demo地址](https://endday.github.io/pwa-vue-cli-demo/#/) 
[项目地址](https://github.com/endday/pwa-vue-cli-demo)

由于笔者水平有限,文中难免有所错误,希望读者朋友不吝赐教,欢迎斧正。
有更好的解决方案可在评论中说明或直接在项目issue中沟通。
