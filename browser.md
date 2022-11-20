# 进程与线程

**进程是 CPU 资源分配的基本单位，线程是 CPU 调度的基本单位**。

进程给线程提供了内存、全局变量，实际上的调度对象是线程，线程是跑在进程里面的，一个进程里面可能有一个或者多个线程。

# 浏览器机制

## 多进程架构

浏览器进程：负责导航栏，书签的前进后退，系统文件读写，网络请求

渲染进程：负责 tab 页面工作，渲染进程负责单个标签页内发生的所有事情。渲染进程里面有：主线程（渲染流水线、JS ）、定时器监听线程、光栅线程、合成线程

插件进程：控制插件

GPU进程：3D 绘制，提高性能

**为什么要多进程？**

1.容错性，多进程互不影响，一个页面崩了不影响其他页面。

2.可以提供安全性。操作系统可以限制每个进程拥有的能力。例如，浏览器限制渲染进程对操作系统读写的能力，只有浏览器进程有读写能力。

3.充分利用 CPU 多个核心

## 网站隔离

浏览器会为每个 tab 分配渲染进程，可是如果一个页面只有一个进程的话，不同站点的 iframe 都会跑在这个进程里面，这也意味着它们会共享内存，这就有可能会破坏同源策略。而进程隔离是隔离网站最好最有效的办法了。所以网站隔离会为网站内跨站点的 iframe 分配一个独立的渲染进程。



# 渲染流水线

## 构建 DOM 树

渲染进程在导航结束后会收到来自浏览器进程提交导航的消息，开始接收数据，主线程也会开始解析并构建 DOM 树。



字节流->字符：根据编码方式将它们转换成各个字符

字符->token：将字符串转换成 token，例如，StartTag、EndTag

token->node 节点 : 对 token 词法分析 转换成 node 节点

node 节点->DOM 树



如果碰到图片，CSS 以及 JS 等资源，主线程会把这些要获取的资源告诉浏览器进程的网络线程。

## 样式计算

主线程根据 CSS 样式计算出每个 DOM 的最终样式

## 布局树 Layout Tree

只知道网页的文档流以及每个节点的样式是不足以渲染出页面内容的，还需要通过布局来计算出每个节点的几何信息。布局的具体过程是：主线程遍历 DOM 树，根据 DOM 的样式计算出**布局树**。布局树上每个节点会有它在页面的x，y坐标以及盒子大小等信息。布局树长得和先前的 DOM 树差不多，不同的是布局树只有可见节点信息。例如如果一个节点被设置为了 display:none，这个节点就不会出现在布局树上面（visibility:hidden 的节点会在布局树上）而不在 DOM 树上。

即使页面的布局十分简单，布局依旧是艰巨复杂的任务。例如页面就是简单地从上而下展示一个又一个段落，这个过程就很复杂，因为你需要考虑段落中的字体大小以及段落在哪里需要进行换行，它们都会影响到的布局。



## 层次树 Layer Tree

通常页面的组成是非常复杂的，如果没有采用分层机制，从布局树直接生成页面的话，那么每次页面有很小的变化时，都会触发重排或者重绘，“牵一发而动全身”会严重影响页面的渲染效率。另外，**各个图层是单独绘制的，互不影响**。

为了确定哪些元素需要放置在哪一层，主线程需要遍历布局树，对特定的节点进行分层，创建一棵层次树。一般情况下，节点的图层会默认属于父节点的渲染层。但也会有单独的渲染层的情况。

**拥有层叠上下文的节点**

1. HTML根元素本身就具有层叠上下文。
2. 元素的定位不是 static 并且设置了z-index
3. 元素的 opacity 值不是 1
4. 元素的 transform 、filter 不是 none
5. 元素的 isolation 值是 isolate

**需要剪裁的地方**

比如一个div，你只给他设置 100 * 100 像素的大小，而你在里面放了非常多的文字，那么超出的文字部分就需要被剪裁。



## 生成绘制列表

主线程根据层次树将图层的绘制拆分成一个个绘制指令，比如先画背景、再描绘边框......然后将这些指令组合成一个绘制列表。



## 合成（切分图块 生成位图）

绘制操作是由合成线程来完成的。绘制列表确定后，主线程就会向合成线程提交这些信息。

因为页面的一层可能有整个网页那么大，一口气生成性能消耗很大，所以合成线程需要将它们切分为图块，然后将图块发送给一系列光栅线程。光栅线程会栅格化每个图块，也就是生成位图数据并且把它们存储在 GPU 的内存中，生成位图的过程实际上都会使用 GPU 进行加速。

合成线程可以给不同的光栅线程赋予不同的优先级，进而使那些离视口近的图块可以先被光栅化。为了响应用户对页面的放大和缩小操作，光栅线程会为不同的清晰度配备不同的图块，并且优先生成低分辨率图块，优化首屏加载速度。

生成完位图数据之后光栅线程发送给合成线程，然后发送给浏览器进程。



## 发送给显卡

浏览器进程中的 viz 组件接收到这个命令，根据这个命令，把页面内容绘制到内存，也就是生成了页面，然后把这部分内存发送给显卡。

每次更新的图片都来自显卡的前缓冲区。而显卡接收到浏览器进程传来的页面后，会合成相应的图像，并将图像保存到后缓冲区，然后系统根据帧率将前缓冲区和后缓冲区对换位置，如此循环更新。

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/15/16f080ba7fa706eb~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)





# 回流

改变 DOM 结构就会触发回流：

- DOM 的几何属性变化，如 width、height、margin
- 增删 DOM 节点导致 DOM 树结构变化
- 浏览器为了获取正确的值，触发回流，如 offset、scroll、client、getComputedStyle()

回流导致整个渲染过程从构建 DOM 树后者样式计算重新走一遍，性能消耗极大



# 重绘

修改 DOM 但没有影响几何属性的时候，会导致重绘（如改变颜色）

从样式计算开始，但是会省去布局、分层的过程，跳到生成绘制列表，然后脱离主线程进行合成。



# 合成层

- CSS3 动画变换 opacity、transform、fliter 等
- 使用 will-change

普通的渲染层被提升为合成层，在样式计算时就跳过了主线程，进入合成阶段。

**直接合成的好处**：

- 不会阻塞主线程，可以运行 JS
- 能够充分发挥 GPU 的优势。因为 GPU 最擅长处理位图数据。



## 隐式合成

当合成层层级太低时，它上下都有渲染层，这就出现交叠问题，合成层的图形上下文被包含在了渲染层的图形上下文里。所以浏览器被迫让合成层上方的所有渲染层提升为合成层，这在大型项目里会导致上千个合成层，直接让页面崩溃，这就是层爆炸。



# 页面性能优化

**加载阶段：缩短白屏时长**

- 内联 CSS 、 JavaScript
- 减少资源个数，发送更少的请求，压缩文件
- 对于不会影响首次渲染的文件加上 async 或 defer



**交互阶段**

减少节流：

- 减少逐项更改样式，直接修改 class 一次性更新
- 进行批量的 DOM 操作使用 createDocumentFragment（文档碎片），然后添加到文档上
- 避免强制同步布局，即多次读取 offset 等几何属性，可以将它们缓存到变量

合成层

- 使用 CSS3 动画，使元素提升为合成层，交给 GPU 处理，GPU 擅长处理位图数据。而如果使用 JS 动画，主线程必须计算每一帧的状态，并不断触发重绘，走完整个渲染流水线。而且主线程任务过多时，会造成动画的延迟卡顿。CSS3 动画更快，也不会阻塞 JS。
- 尽量让动画节点的 z-index 高一点，避免隐式合成。



# 浏览器安全

同源策略：两个 URL 的**协议、域名和端口**都相同，我们就称这两个 URL 同源

两个相同的源之间可以

- 相互操作 DOM
- 读取 Cookie、IndexDB、LocalStorage 等数据
- 允许 XHR 请求



## XSS

XSS，跨站脚本攻击。指注入恶意脚本。

恶意脚本会：

- 窃取 Cookie 等信息
- 监听用户行为，如监听用户输入的银行卡等信息
- 修改 DOM 伪造窗口，欺骗用户输入用户名和密码等信息，或者在页面内生成浮窗广告

**存储型 XSS**

黑客利用站点漏洞将恶意代码提交到网站的数据库中，用户向网站请求包含了恶意脚本的页面，浏览该页面的时候，恶意脚本就能执行。

例子：永远不要相信用户的输入

**反射型 XSS**

恶意 JavaScript 脚本属于用户发送给网站请求中的一部分，随后网站又把恶意 JavaScript 脚本返回给用户。

例子：通过 QQ 群或者邮件诱导用户去点击恶意链接

**基于 DOM 的 XSS**

基于 DOM 的 XSS 攻击是不牵涉到服务器的。黑客在 Web 资源传输过程或者在用户使用页面的过程中用路由器或恶意软件修改页面数据。

存储型 XSS 攻击和反射型 XSS 攻击都是服务端漏洞。而基于 DOM 的 XSS 攻击是客户端漏洞。

**如何防范**

无论怎样 XSS 都是利用恶意脚本注入实现的。

- 服务器对关键字符过滤或转码
- 实施内容安全策略，限制加载其他域的文件，禁止向第三方域提交数据，禁止执行未授权脚本
- 使用 HttpOnly 属性。很多 XSS 都是盗用 Cookie 的，因此可以设置 HttpOnly，让 JS 读取不到。



## CSRF

全称是跨站请求伪造。CSRF 攻击指的是黑客诱导用户点击链接，打开黑客的网站，然后利用用户的登录状态发请求。

比如点击链接后：

**自动发 GET 请求**

进入页面后自动发送 get 请求，这个请求会自动带上关于你的 cookie 信息。假如服务端没有相应的验证机制，它可能认为发请求的是一个正常的用户，因为携带了相应的 cookie，然后进行相应的各种操作，可以是转账汇款等恶意操作。

**自动发 POST 请求**

黑客可能自己填了一个表单，写了一段自动提交的脚本，同样带上 cookie，让服务器误以为是一个正常的用户在操作。



这就是 CSRF 攻击的原理。和 XSS 攻击对比，CSRF 攻击并不需要将恶意代码注入用户当前页面，而是跳转到新的页面，利用服务器的**验证漏洞**和**用户之前的登录状态**来模拟用户进行操作。

所以我们主要对服务器下功夫：

- **利用 Cookie 的 SameSite 字段**，SameSite 可以设置为三个值，Strict、Lax、None。在 Strict 模式下，完全禁止第三方请求携带 Cookie。Lax 宽松一点，只能在 get 方法提交表单或者a 标签发送 get 请求的情况下可以携带 Cookie。none 的话请求自动携带 cookie。
- 验证来源站点，用到请求头中的两个字段: Origin、Referer。Origin 只包含域名，而 Referer 包含了具体的 URL 路径。但可以自定义请求头伪造，没什么大用。
- **CSRF Token**。在浏览器向服务器发起请求时，服务器生成一个 CSRF Token，然后将该字符串植入到返回的页面中。在浏览器端发起请求需要带上页面中的 CSRF Token，然后服务器验证。第三方页面上是没有这个 Token 的。



# 前端缓存

## Service Worker

Service Worker 出现较晚，使用不算广泛。

Service Worker 借鉴了 WebWorker 思路，即让 JS 运行在主线程之外，是服务端和客户端之间的中间层。在 Service Worker 中使用 cache 相关的 API 可以实现缓存。



## memory cache

memory cache 是浏览器为了加快读取缓存速度而进行的自身的优化行为。

memory cache 生命周期很短，与渲染进程（页面）一致。优势是效率最快，因为就在内存里。

而 HTTP 缓存存储在磁盘中，比内存缓存慢。优势在于存储容量和时长。

- 比较大的文件丢进磁盘，反之丢进内存
- 内存使用率比较高的时候，优先放入磁盘



## HTTP缓存

协商缓存的两个字段都需要配合强缓存中 Cache-control 字段来使用，只有在未能命中强制缓存的时候，才能协商缓存。

### 强缓存

强缓存不需要发送 HTTP 请求

- `Cache-Control`， 是一个相对时间；
- `Expires`，是一个绝对时间；

优先使用 Cache-Control。 因为服务器的时间和浏览器的时间可能并不一致，且 Cache-control 选项更多，设置更精细。

两者同时存在时，Cache-Control 的优先级高于 Expires 。

```js
Cache-Control:max-age=3600
```

- s-maxage：针对代理服务器的缓存时间
- public: 客户端和代理服务器都可以缓存。
- private：只有浏览器能缓存
- no-cache：跳过强缓存，直接进入协商缓存阶段

### 协商缓存

协商缓存（即：304）：服务端告知客户端是否可以使用缓存

 `If-Modified-Since`  、`Last-Modified`

-  `Last-Modified`：标示资源的最后修改时间；
-  `If-Modified-Since`：如果最后修改时间大，说明资源被改过，则返回最新资源，HTTP 200 OK；如果最后修改时间小，说明资源无修改，响应 HTTP 304 协商缓存。

`If-None-Match` 、 `ETag` 

-  `Etag`：唯一标识
-  `If-None-Match`：将 If-None-Match 值设置为 Etag 的值。服务器收到请求后进行比对，如果资源没有变化返回 304，如果资源变化了返回 200。

优先使用 Etag，因为 Last-Modified 有问题：

- 编辑了资源文件，但是文件内容并没有更改，这样也会造成缓存失效。
- Last-Modified 能够感知的单位时间是秒，如果文件在 1 秒内改变了多次，那么这时候的 Last-Modified 并没有体现出修改了。

![img](https://img-blog.csdnimg.cn/d92026ce085b401c95cf02b7ce9b7fae.png)

## Push Cache

推送缓存，这与 HTTP/2 中的服务端推送有关，应用的并不广泛。



# PWA

PWA，全称渐进式网页应用。即渐进式的过渡方案，让普通站点逐步过渡到 Web 应用，以降低站点改造的代价，使得站点逐步支持各项新技术，而不是一步到位。不是动不动就是取代 App、小程序这些本地应用。

PWA 解决方案

- 网页依赖于浏览器，所以需要一级入口
- 弱网环境无法使用，没有消息推送

PWA 提供 manifest.json 配置文件，可以让开发者自定义桌面图标、显示名称、启动方式等信息，用于一级入口。

**Service Worker **解决离线存储和消息推送。在页面和网络之间增加一个中间层，用来缓存和拦截请求。有 Service Worker 之后，网页请求资源时，会先通过 Service Worker，让它判断是返回 Service Worker 缓存的资源还是重新去网络请求资源。一切的控制权都交由 Service Worker 来处理。另外，Service Worker 是运行在浏览器进程中的，生命周期依附于浏览器，即使页面没开，他也能接收服务器推送。

PWA 从商业和技术角度来看都没法进入移动端，PC 其实还可以。移动端 APP、小程序商业生态已经成熟，APP 可以尽情偷用户数据，公司肯定不乐意用对用户更安全的网页。另外 PWA 技术发展的路太长，国内市场已经被攻占了。