# 谈谈项目中的亮点难点

++避免功能性问题 & 业务型问题，除非它真的很复杂，很有亮点

++重点讲解的是：

​		++性能优化方案

​			层面：webpack / HTTP /页面渲染/异步加载/大数据渲染优化

​			强调结果，例如：原来打包/加载时间是N秒，经过我的优化后是M秒

​		++插件组件封装[敏捷化平台构建之一]

​			公共组件库/插件组件二次封装/ vue 自定义指令

​			除强调结果外，还可以突出自己的原理/源码阅读能力



# 跨域

跨域导致：

- Cookie、LocalStorage、IndexedDB 不可被访问
- 不可操作 DOM
- AJAX 请求结果会被拦截

## JSONP

利用 script 标签没有跨域限制的漏洞，可以得到其他源的数据。回调要声明在全局，后端接收到请求把收到的函数名和数据拼成字符串，如：“callback（data）”。客户端收到之后就能执行回调。

优缺点：简单兼容性好，但只支持 get 请求（因为 script img link 都是去加载资源的），安全性不好（可能劫持url 返回个木马程序就直接执行了）

```js
function jsonp({ url, params, callback }) {
    return new Promise((resolve) => {
        let script = document.createElement('script')
        let paramsArr = []
        for (let key in params) {
            paramsArr.push(`${key}=${params[key]}`)
        }
        paramsArr.push(`callback=${callback}`)
        script.src = `${url}?${paramsArr.join('&')}`
        document.body.appendChild(script)
        window[callback] = (data) => {
            resolve(data)
        }
    })
}
jsonp({
    url: '',
    params: {},
    callback: ''
}).then(data => {
    console.log(data)
})
```



## CORS

CORS 关键在于后端。服务器在头信息里设置 Access-Control-Allow-Origin 即可规定允许跨域的源。

缺点是设置允许的源要么只能设置一个，要么就都允许（但不允许带 cookie 因为不安全）

另外，服务器通常会设置在正式通信之前，增加一次预检请求，用的是 option 方法，以此知道服务端是否允许跨域请求。



## Node中间件代理

同源策略是浏览器的标准，服务器向服务器请求不需要。

只需要用 node 中间件实现代理服务器来转发请求和响应即可。



## nginx 反向代理

实现原理类似于 Node 中间件代理，需要搭建一个中转 nginx 服务器用于转发请求。实现方式简单，只需要改改配置，并且支持所有浏览器。



## Websocket

Websocket 是 HTML5 的一个全双工的持久化协议。建立连接时需要借助 HTTP，建立完成 client 与 server 的双向通信就与 HTTP 无关了，也不用发 ajax 了。只不过长连接比较消耗性能。



## postMessage

H5 的 API，用于跨窗口、嵌套的 iframe 通信



## window.name + iframe

window.name 的独特之处：name 值在不同的页面、不同的域名加载后依旧存在。

通过 iframe 的 src 属性由外域转向本地域，跨域数据即由 iframe 的 window.name 从外域传递到本地域，这就巧妙地绕过了浏览器的限制。



## location.hash + iframe

a.html 欲与 c.html 跨域通信，通过中间页 b.html（a、b 同域）来实现。 三个页面，不同域用 iframe 的 location.hash 传值，相同域直接 js 通信。



## document.domain + iframe

只能用于二级域名相同的情况，比如 `a.test.com` 和 `b.test.com` ，只需要给页面添加 `document.domain ='test.com'` 表示二级域名相同就可以实现跨域通信。



## 总结

跨域的解决方案思路两种，绕过去和 CORS。

CORS 算是真正的跨域解决方案，因为双方通过预检请求协商后允许跨域。

JSONP 优势在于支持老式浏览器，以及可以向不支持 CORS 的网站请求数据。

nginx 反向代理是比较完美的方案，用的很多。

各种 iframe 的方式组织和控制代码逻辑太复杂，鸡肋。



# cookie/session/token/jwt

为了解决 HTTP 无状态，有了 cookie。cookie 是由服务器发送给客户端的，用来记录用户状态。浏览器再向服务器请求时会自动携带 cookie。cookie 的好处是前端无感知，自动参与双方通信，是鉴权的基石。

session 是服务端保存状态，客户端只存储 sessionId。

cookie 只是数据载体，sessionId 是传递的数据。cookie 是客户端存储方式，session 是服务端存储状态。

session 的维护给服务端带来很大压力。

token，令牌，使用 token 通信每次服务端只需要核对令牌有效性。token 保存在客户端。

jwt 就是制定了 token 的标准。JWT 是一个字符串，中间用 . 分割成三部分，头部、载荷、签名。头部是 jwt 的元信息，主要是使用的签名算法，然后把 JSON 用 Base64URL 算法转成字符串。载荷就是用户信息、有效时间等，同样转成字符串。Signature 是保证安全性，防止数据篡改。

一般认为，session 是「种在 cookie 上、状态存在服务端」的认证方案，token 是「客户端存哪都行、数据存在 token 里」的认证方案。token 的问题在于已经签发的不大好收回。



# 竞态问题

- 有一个分页列表，快速地切换第二页，第三页；
- 先后请求 data2 与 data3，分页器显示当前在第三页，并且进入 loading；
- 但由于网络的不确定性，先发出的请求不一定先响应，所以有可能 data3 比 data2 先返回；
- 在 data2 最终返回后，分页器指示当前在第三页，但展示的是第二页的数据。

这就是竞态问题，在前端开发中，常见于搜索，分页，选项卡等切换的场景。



## 取消请求



### XHR 取消请求

使用 abort 方法

```js
const xhr= new XMLHttpRequest();
xhr.open('GET', 'https://xxx');
xhr.send();
xhr.abort(); // 取消请求
```



### fetch 取消请求

要中止 fetch 发出的请求，需要使用 `AbortController`，但取消请求的 API 还在实验阶段。

```js
const controller = new AbortController();
const signal = controller.signal;

fetch('/xxx', {
  signal,
}).then(function(response) {
  //...
});

controller.abort(); // 取消请求
```



### axios 取消请求

调用 axios 的取消请求 API 时，内部调用了 Promise.reject（）与 xhr.abort（）/ fetch 的 AbortController

> 毕竟取消请求也视作请求失败！

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ac9450b1d15475abf70977f7bb17c3c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)



## 忽略请求

忽略请求的本质是让 promsie 一直 pending，即忽略请求结果，看上去的“取消请求”。

指令型 promise 和 id 的方式都是在封装高阶函数。

### 指令型 promise

高度控制权的 promise：在外部就可以让 promise 成功、失败、取消

```js
function classPromise() {
  let prePromise = null;
  function wrapPromise(fn) {
    prePromise && prePromise.cancel();// 取消上一次的请求
    let res, rej, cancel;
    let promise = new Promise((resolve, reject) => {
      res = resolve;
      rej = reject;
      cancel = () => {
        resolve = null;
        reject = null;
      }; // 让 promise 一直 pending,执行不了回调
      setTimeout(() => {
        resolve && resolve("data,Yes");
      }, 3000);
    });
    prePromise = promise;
    promise.res = res; // 暴露接口
    promise.rej; // 暴露接口
    promise.cancel = cancel; // 暴露接口
    return promise;
  }
  return wrapPromise;
}
let wrapPromise = classPromise();
let promise1 = wrapPromise(() => {});
promise1.then((data) => {
  console.log(data);
});
let promise2 = wrapPromise(() => {});
promise2.then((data) => {
  console.log(data);
});
promise2.res(111);
```



### 通过 id 标识请求

```js
function classPromise() {
  let id = 0;
  function wrapPromise(fn) {
    let uid = ++id;
    let promise = new Promise((resolve, reject) => {
      setTimeout(() => {
        if (uid === id) resolve && resolve(uid);
        // 是最新的 id（请求）才接收结果
      }, 3000);
    });
    return promise;
  }
  return wrapPromise;
}
const wrapPromise = classPromise();
let promise1 = wrapPromise(() => {});
promise1.then((data) => {
  console.log(data);
});
let promise2 = wrapPromise(() => {});
promise2.then((data) => {
  console.log(data);
});
```

## 总结

「取消」更实际

如果请求被「取消」了没有到达服务端，那么可以一定程度减轻服务的压力。

但是取消请求也依赖底层的请求 API，比如 XMLHttpRequest 需要用 abort，而 fetch API 和 axios 需要用 AbortController。

「忽略」更通用

而「忽略」的方式，不依赖请求的 API，更加通用，更容易抽象和封装。本质上所有的异步方法都可以用忽略的方式取消请求。

一个更实际，一个更通用，两者的使用需要根据具体场景来权衡。

取消方案：不让竞态问题产生，数据没请求回来的时候 tab 直接禁用或者直接全屏 loading 不让点击。（有局限性，输入框上的竞态问题不能用）



# 工程化

## 解决的问题

1. 语言编译（ES6+ 语法、TypeScript、Sass）
2. 模块化（ES Modules）
3. 重复的机械式工作（构建工具、持续集成部署）
4. 代码管理，代码风格（代码托管 git、ESLint、prettier）



## 工程化不等于工具

有些工具非常强大，像 Webpack，以至于很多人认为前端工程化就是指 webpack。

webpack 是工程化工具的集成，工具不是工程化的核心，工程化的核心是对项目开发的整体规划和架构，工具只是实现这种规划和架构的一种手段。



## 发展历史

### 任务运行器

工程化首先要做的就是支持各种代码的编译。最早的前端工程化是通过任务的形式组织这些编译过程的，指定对什么文件用什么编译器编译，然后输出到哪个目录。任务之间可以规定先后顺序、串行并行。gulp 就是这一类工具，叫做任务运行器。

这一类工具能够组织整个编译流程，对不同的文件分别做相应的处理。但因为每个任务都比较独立，很难做一些全局的优化。

### 打包工具

后来出现了另一种思路，不通过任务组织了，而是分析模块之间的依赖关系，从入口模块开始构建依赖图，中间遇到的用到的所有模块都会作为它的依赖。然后对依赖图的每个节点分别用对应的编译器处理。

这个和 task runner 的方式有啥区别？不都是对不同的文件用不同的编译器处理么？

现在有了模块之间的依赖图了，那就可以做一些**全局优化**：

比如通过分析依赖关系来去掉一些没有用到的代码：tree shaking

比如把这些模块拆分到不同的 chunk（分组）里，这样把变动频繁的模块和不咋变动的模块分到不同的 chunk，进而生成到不同的文件里，就可以更好的利用缓存，即分包。

而且，因为生成的代码是自己控制的，有自己的运行时代码，那就可以实现模块的懒加载，也就是把 code splitting 分出来的 chunk，在运行时动态加载。

但是打包工具也不是完美的，因为每次都要构建整个依赖图，对不同文件分别做处理，之后才能生成代码。所以当项目的模块多了就会很慢。

### no bundle

不打包也就不会进行依赖分析，那怎么确定处理哪些文件呢？

no bundle 是基于浏览器支持 ESmodule 来实现的，浏览器会做 ESmodule 的依赖分析，然后加载对应的模块，这样自然就不用自己做依赖分析了，只需要实现模块的编译即可。

当然，生产环境还是需要打包的，会用打包工具来处理。no bundle 方案只是解决了开发环境下打包工具要构建整个依赖图导致比较慢的痛点问题。

### 总结

构建的核心是对不同的文件做不同的编译，最早的任务运行器的方案实现了编译流程的组织，但是并没有做全局的优化，也没有自己的 runtime 代码。所以出现了基于依赖分析的打包工具，打包工具可以基于依赖分析实现 treeshaking、code splitting 等优化，可以配合 runtime 代码实现懒加载。但成也依赖分析，败也依赖分析，这个太慢了，所以出现了 no bundle 的方案，配合浏览器对 es module 的支持，只要实现对应模块的编译服务即可，不过生产环境还是要打包的。



# monorepo

简单来说就是将多个项目整合到了一个仓库里来管理，packages 文件夹下存在一堆文件夹，这每个文件夹都对应一个包。



## multirepo 优劣

> 仓库体积小，模块划分清晰

> 多仓库来回切换（编辑器及命令行），项目一多真的得晕。如果仓库之间存在依赖，还得各种 npm link , npm link 时不同项目的依赖可能会存在冲突问题。

> 代码复用差

有一些逻辑很有可能会被多次用到，比如公共组件、工具函数，或者一些配置。如果复制多份，这些代码出现 bug、或者需要做调整的时候，就得修改多份，维护成本越来越高。

那如何来解决这个问题呢？比较好的方式是将公共的逻辑代码抽取出来，作为一个 npm 包进行发布，一旦需要改动，只需要改动一份代码，然后发布就行了。然后每个项目都安装新版本包。

因为不同的仓库工作区的割裂，导致复用代码的成本很高，开发的流程繁琐，尤其在基础库频繁改动的情况下体验很差。

> 项目基建成本高

各个项目的工作流是割裂的，因此每个项目需要单独配置开发环境、配置集成部署、发布流程。

并且如果项目间存在构建、部署和发布的规范不统一的情况，这样维护起来就更加麻烦了。



## monorepo 优劣

共同依赖可以提取至 root，版本控制更加容易，依赖管理方便

基建成本低，所有项目复用一套标准的工具和规范，无需切换开发环境，如果有新的项目接入，也可以直接复用已有的基建流程，比如 CI 流程、构建和发布流程。

团队协作也更加容易，都在一个仓库开发，能够方便地共享和复用代码，方便检索项目源码。

> 项目如果变的很庞大，那么 git clone、构建都会是一件耗时的事情



# vite

不打包，按需加载。适用于开发环境，基于 ESModule。

启动一个 koa 开发服务器，并劫持浏览器的 HTTP 请求，在后端进行相应的处理返回给浏览器。

解析 import ，不过不是 ./ 或者 / 开头的就重写路径加上 @modules，改了路径资源就找不到了，所以要建立映射表重写内容并返回，在这过程中也要处理一下 process 不存在的问题。遇到 .vue 文件编译成 render 函数并缓存。整体核心就是解析路径返回处理后的模块。

Webpack 启动后会做一堆事情，经历一条很长的编译打包链条，从入口开始逐步经历语法解析、依赖收集、代码转译、打包合并、代码优化，最终将高版本的、离散的源码编译打包成低版本、高兼容性的产物代码，这可满满都是 CPU、IO 操作啊，在 Node 运行时下性能必然是有问题。

而 Vite 运行 Dev 命令后只做了两件事情，一是启动了一个用于承载资源服务的 service；二是使用 esbuild 预构建 npm 依赖包。之后就一直躺着，直到浏览器以 http 方式发送模块请求时，Vite 才开始按需编译被请求的模块。

除了启动阶段跳过编译操作之外，Vite 还有很多性能优化：

- 预构建：Esbuild 在预构建阶段先打包整理好，将 commonJs 的文件提前处理，转化成 ESM 模块。并且合并内部模块，减少 http 请求数量。
- 客户端强缓存：请求过的模块会被设置为强缓存，如果模块发生变化则用附加的版本 query 使其失效
- 内置更好的分包实现：不需要用户干预，默认启用一系列智能分包规则，尽可能减少模块的重复打包
- 产物优化：打包产物更小，没有模板代码没有运行时



打包不是目的，运行才是，能交给浏览器做的事情就交给浏览器吧。

越容易上手的工具往往意味着越难被定制，如果只是在 Vite 预设好的边框里面玩确实很容易。vite 内置好各种业界最佳实践，没有太多定制空间。



# 性能优化

- 资源本身大小的压缩优化（想办法减少资源的体积）
- 网络请求的全过程
- 浏览器渲染的全过程

## 页面内容

减少请求数量：合并 JS 文件，雪碧图等

延迟加载：非首屏使用的脚本、样式、图片都可以延迟加载

预加载：load 事件触发后马上加载，或者用户做出某种行为就开始预先加载

划分到不同域名：浏览器对用一个域名的并行下载量有限制，可以把资源分配到不同域名，但不能太多，避免 DNS 查询的消耗

## 服务器

CDN 加速

设置合适的 HTTP 缓存

开启 Gzip（图片、pdf 除外，本身就压缩过，开启会浪费资源且增加文件体积）

## CSS

把样式表放在 `<head>` 中：如果把样式表放在页面底部，一些浏览器为减少重绘，会在 CSS 加载完成以后才渲染页面，用户只能对着白屏干瞪眼，用户体验极差。

## JS

把脚本放在页面底部，也可以考虑 script 的 defer 、async 属性

避免强制抖动布局等，文档碎片

## 图片

图片懒加载

图片压缩，base64