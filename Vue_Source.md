# MVC MVVM

- 传统的 MVC 指的是,用户操作会请求服务端路由，路由会调用对应的控制器来处理,控制器会获取数 据。将结果返回给前端,页面重新渲染
- MVVM :传统的前端会将数据手动渲染到页面上, MVVM 模式不需要用户收到操作 dom 元素,将数据绑定到 viewModel 层上，会自动将数据渲染到页面中，视图变化会通知 viewModel层 更新数据。 ViewModel 就是我们 MVVM 模式中的桥梁

> Vue并没有完全遵循MVVM模型，严格的MVVM模式中,View层不能直接和Model层通信,只能通过ViewModel来进行通信。



# Vue2

## 响应式

数据驱动视图，所以我们首先要能够侦测变化，Vue 2 中 defineProperty 改写属性，以此侦测到数据变化。然而我们还需要知道哪里使用到了数据，这样数据变了才能通知它。所以我们要先在 getter 中收集依赖，然后等属性变化时，在 setter 中通知依赖。我们封装 Dep 类用来收集、删除、通知依赖。所谓的依赖就是 watcher。依赖收集时 watcher 先把自己设置到一个全局变量，watcher 读取了属性值，就走到 getter，在 getter 中就会从全局变量拿到 watcher 收集到 Dep 中，这样 watcher 就订阅了数据了数据的变化。

数组追踪变化的方式与对象不同，因为我们通常用数组方法改变数组，而且如果改写数组中所有内容太浪费性能，所以我们通过创建拦截器也就是伪造原型重写七个方法的方式来追踪变化。如果连原型都不支持，我们直接把拦截器上的方法设置到数组身上。数组收集依赖的方式和对象一样，都是在 getter 中收集。但是由于数组要在拦截器里向依赖发送消息，所以依赖不能像对象那样直接保存在 defineReactive 中，而是保存在 Observer 实例上。

Observer 是响应式数据的入口，我们还要为已经侦测变化的数据加上 __ ob __ 的标记，并把 this 也就是 Observer 实例放到 __ ob __ 上。一方面可以保证一个数据只被侦测一次，一方面可以通过这个属性拿到 Observer 上保存的依赖。注意的是，这个属性要不可枚举，否则会陷入死循环。

当然除了被侦测的数据本身之外，我们还要递归侦测对象或数组中的数据。除了侦测已有的数据之外，如果是用push、unshift、splice 方法新增了数据也要侦测。

到这里只实现了 Dep 收集 Watcher，Watcher 同样也需要知道自己订阅了哪些 Dep，这样当 Watcher 想取消订阅这些 Dep 时能通知他们把自己从依赖列表里移除。收集 Dep 时，我们需要获取 Dep 的 id，否则数据每次更新 Watcher 都会重新读值导致 watcher 被重复收集。这样就是 watcher 自己设置到全局变量，读值，dep 收集 watcher， watcher 收集 dep。此时就完成了 dep、watcher 相互收集，它们是多对多的关系。

由于 ES6 以前，JS没有提供元编程的能力，受限于 defineProperty，对象里直接新增属性或是用 delete 删除属性都不能被拦截，数组改变长度或是用下标修改数据也是一样。

### vm.$watch

只是对 watcher 的封装，和渲染 watcher 大差不差，核心就一句话 new Watcher() 就完事了。

deep 实现原理：deep 除了要触发当前监听数据的依赖收集、派发更新之外，还要把当前监听的这个值的所有子值都触发一遍收集依赖逻辑。只需要增加递归的操作，让所有的子值 dep 都能收集到当前 watcher 即可，唯一需要注意的是一定要在全局变量被设置为 undefined 前去触发子值的依赖收集工作。

immediate：只需要立即执行 handler 函数即可

### vm.$set

往数组上新增就帮你调 splice，往对象上新增就调 difineReactive 改写成 getter、setter 即可。

### computed

计算属性默认不执行。多次取值，依赖的值不变化就不会重新执行。

计算属性本质就是一个 watcher。我们用 lazy 标志位区分用户 watcher 和 computed，lazy 就说明默认不执行 getter。dirty 标志位表示这个值是不是脏的，默认为 true，因为第一次取值肯定要执行。取值后变为 false，依赖的值变化了，dirty 变为 true，缓存就是依赖于 dirty 实现的。依赖变了才会执行 getter 取值。

流程：我们上来是先渲染，所以全局变量 target（也就是栈顶）是渲染 watcher，组件要取计算属性，所以 target 就变成了计算属性 watcher，计算属性求值，计算属性依赖的数据就可以收集计算属性 watcher，然后弹栈发现还有个渲染 watcher，于是渲染 watcher 就通过计算属性 watcher 在它的 deps 列表里拿到它的依赖，这样依赖也能收集到渲染 watcher。这样数据变了，就通知计算属性，计算属性把 dirty 变成 true，由于数据同时收集了渲染 watcher 和计算属性，渲染 watcher 也得到通知发现 dirty 变了便重新计算一次计算属性再渲染。

## Vue.nextTick

### 异步更新策略

数据变化通知渲染 Watcher 更新，但如果组件里两个数据变了导致组件更新两次是不合理的。要解决这个问题，我们可以将收到通知的 watcher 都暂存到队列里，并且在添加之前先检查是否存在相同的 Watcher。然后在下一次事件循环里，我们让队列的 watcher 触发渲染流程并清空队列。这样就保证了每个 watcher 只重新渲染一次。

所以总结下来，nextTick 核心是批处理的思想。首次调用 nextTick 时，我们把回调加到队列里，然后发现 pending 标志位为 false，说明我们这是本轮事件循环第一次使用 nextTick，我们就封装一个异步操作把冲刷队列的操作丢到微任务队列里，这内部有层层的兜底操作（Promise，MutationObserver，setImmediate，setTimeout）清空队列后我们重新把标志位置为 false。（标志位是因为一次事件循环里我们只需往微任务队列里添加一次冲刷队列的操作即可，有点防抖的意思）

## 生命周期

初始化 options（合并配置）

初始化实例属性（赋一些初始值），

初始化事件（父组件在模板中使用 v-on 监听子组件内触发的事件），

### beforeCreate

初始化 inject。

初始化 props、methods、data、computed、watch（这样 data 可以使用 props，watch 可以观察 props、data）

初始化 provide

### created

把模板编译成渲染函数

### beforeMount

挂载，把模板渲染到指定的 DOM 元素中，并开启依赖追踪

### mounted

已挂载阶段，持续追踪状态变化

### beforeUpdate

重新渲染

### updated

调用 vm.$destroy

### beforeDestroy

卸载依赖追踪、子组件、移除事件监听器

### Destroyed



## patch 逻辑

patch 有三个作用：首次渲染、更新、销毁

- 如果老的 VNode 是真实元素，则表示首次渲染，创建整棵 DOM 树，并插入模板

- 如果新的 VNode 不存在，老的 VNode 存在，则调用 destroy，销毁老节点

- 如果老的 VNode 不是真实元素，并且新的 VNode 也存在，则表示更新阶段，执行 patchVnode

  - 首先全量更新当前节点所有属性

    如果新老 VNode 都有孩子，则递归更新孩子节点，进行 **diff** 过程

    **针对前端操作 DOM 节点的特点进行如下优化：**

    - 同层比较：一层一层去做比较，因为在dom修改里会很少涉及跨层次的修改，更多的是同一层次的增删改查，这是 diff一个巧妙的地方
    - 针对前端操作数组都是 push shift 等对头尾操作，不会完全打乱的情况：Vue2 采用双端diff ，即 while + 头尾指针，直到新旧有一方的头指针超过尾指针结束循环
    - 先进行新头新尾旧头旧尾的四次比对，如果命中，就不需要暴力比对，减少了节点的移动次数。如果没有命中，就说明乱序了，开始暴力比对
    - 比对前先建立 key-index 的映射表，找到了就能复用，没找到就新建
    - 比对结束后如果老 VNode 先于新 VNode 结束，则新增剩余节点；反之删除多于节点

### key

不写 key 和把 index 作为 key 都是一样的，我们都把这两个节点当作同一节点（key、tag 相同）判断 key 时 undefined = undefined。就导致本不是相同节点我们当作相同节点，然后错误的复用，产生不必要的更新与性能浪费。除了性能外，如果结构中包含输入类的 DOM，会产生错误的 DOM 更新。



# Vue3

## 响应式

响应式系统的实现依赖于对数据读取的更改的拦截以及让数据和副作用函数之间建立联系。取值时把 effect 放到桶里，更改时让桶里的 effect 重新执行。接着我们构建一个完善的桶结构，weakMap 由 target-Map 组成，map 由 key-Set 组成，当用户代码对一个对象没有引用关系时，weakMap 不会阻止垃圾回收器的回收。我们还需要在每次 effect 重新执行前，先把它从与之关联的依赖集合中删除，这样每次执行会重新收集依赖，这样才能解决 effect 依赖的数据变化导致不必要更新的问题。effect 想要把自己从依赖集合中删除，自己也要知道那些集合依赖它，所以也是互相收集。然而在实现删除 effect 时，发现 Set.forEach 在删除属性又添加属性时会无限运行下去，所以我们会先拷贝一份 set ，再去遍历新的 set ，这样就不会无限执行。effect 嵌套（父子组件）问题我们依旧用栈解决（也可以给 ReactiveEffect 类添加 parent 属性）。

### reactive

使用 proxy 对对象实现响应式

1 实现同一个对象代理多次，返回同一个代理 （缓存，将代理完成的 proxy 放到 weakMap 里）

2 代理对象被代理，直接返回此代理对象 （因为如果是代理对象，会走到 getter，只有代理对象才有getter，所以可以判断为 proxy ，直接返回）

为什么要用 Reflect ？

```js
const obj={
    a:1,
    get b() {
        return this.a // this 指向obj
    }
}
```

如果不用 Reflect 改变 this，我们只收集到 b，没收集到 a。

当读取 b 时，reflect 相当于改变 this 指向（proxy），使得 effect 与数据建立联系，达到依赖收集的目的。

### computed

```js
effect(()=>{
    computed
})
computed(()=>{
    a,
    b
})
```

计算属性本身是一个 懒执行的 effect 。

1 a，b 要依赖于计算属性 effect （因为 computed 用到了 a，b）

2 计算属性收集了外层 effect （组件 effect）
  依赖的值（a,b）变化了会触发计算属性effect 重新执行，计算属性重新执行的时候会触发外层 effect 来执行

dirty ：计算属性肯定要有一个缓存的标识，，缓存依赖于 dirty ，默认是脏的，只有依赖有变化 dirty 才会变脏，才重新执行 get 取值

实现方式是 effect + 调度器 ，computed 充当垫片。

取值时进行依赖收集，computed 收集外层 effect （computed effect 也有 dep），然后 computed effect 执行

外层 effect -> computed effect -> 属性 -> computed effect -> 外层 effect (外传内，内传外)

### watch

同样是 effect + 调度器

把观测的数据包装成 effect 函数 ，包装时要递归对象进行依赖收集，同时用调度器包装用户的回调 cb 。然后手动调用 effect ，拿到 oldValue ，观测的数据变化时触发调度器执行时会重新执行 effect 拿到 newValue，然后传给 cb 执行。

### ref

**本质是包裹对象**，把原始值包裹成对象，然后用 reactive （obj）实现

并为包裹对象添加一个不可枚举的属性 _v_isRef 用来区分 ref 与 reactive

### 处理响应丢失问题

扩展运算符导致响应丢失问题：

```js
let obj = {
    a:1,
    b:2
}
return {
    ...obj
}
```

**toRef ：做一层访问代理**，将数据转换为类似于 ref 结构的包裹对象

```js
function toRef(obj,key){
    return {
        get value(){
            return obj[key]
        }
    }
}
```

toRefs：遍历对象，每个属性都用 toRef（obj，key）

ref 带来了新的问题：要一直 .value

### 脱 ref

自动脱 ref：我们不希望去 ref.value

**用 Proxy 创建一个代理对象**，如果值有 _v_isRef 属性说明它是 ref，那就**直接返回 .value**

## 模板编译

编译器的目标：解析模板（parce（str））得到模板 AST ，将 AST 转换（ transform ）成 JS AST ，根据 JS AST 生成（generate（jsAST）） 渲染函数 ( render )。

```js
//渲染函数：
function reder(){
    return h('div',[
        ...
    ])
}
```

AST 就是一个与模板同构的对象，用来描述 html 节点。

我们需要将模板 AST 转换成（ transform ）用于描述渲染函数的 JS AST。

而后生成 render 函数 ：const code = generate（jsAST）

### 解析器 parser

```js
function parser( template ){
    ...
}
```

```html
<p>文本</p>

三个token：
开始标签：<p>
文本节点：文本
结束标签：</p>
```

参数为模板字符串，解析器从前向后读取，用正则不断匹配并切割为一个个 token（词法记号）（比如开始结束标签、注释、本文等）。而后我们根据 token 列表构建 AST。从第一个 token 开始，顺序扫描整个 token 列表。在这个过程中，我们需要用一个栈来维护元素间的父子关系。每遇到一个开始标签节点，我们就构造一个 AST 节点并压栈。遇到结束标签，就将栈顶元素弹出。以此保证栈顶元素始终是父节点。扫描时遇到的所有节点都会被添加到栈顶节点的 children 属性下。扫描完所有 token 后，AST 就构建完成了。

### 转化 transform

Vue 2 diff 的特点是递归，每次都要比较每一层，全量比对，只是为静态节点做了标记遇到了就跳过

所以需要转化：对动态节点做标记 patchFlag（有动态的属性如指令、插槽、事件就是动态节点），并用 Block 收集动态节点，以及所有动态子代节点， 将树的递归拍平成一个数组（递归变成 for 循环），并且只更新必要的属性，实现了靶向更新。当然如果是 v-if / v-else / v-for 这种能破坏 DOM 结构的就只能递归全量比对。

性能方面：

- 为静态节点与动态节点设计不同的 transform 函数，还进行组合（组合表达式）。这种组合的目的是为了创建或者更新这类 VNode 时进行更少的操作（合并）。

  ```html
  <div> {{a}} 123 </div>
  ```


- 静态提升：响应式数据变化时整个渲染函数会重新执行，这会导致静态的虚拟节点更新时也会被创建一次，所以我们把纯静态的虚拟节点提升到渲染函数之外，导致静态节点在整个生命周期中只会创建一次
- 预字符串化：基于静态提升，假如模板中包含大量连续的静态标签节点，我们便将它们序列化为字符串，生成一个静态类型的虚拟节点。这样就减少了创建虚拟节点的性能开销和内存占用。
- 缓存内联事件处理函数：对于绑定事件的组件，而且是内联语句，编译器会为其创建一个内联事件处理函数。每次重新渲染时都会为组件创建全新的 props 对象，同时 props 中的函数也会是全新的，这造成了额外的开销。只需给渲染函数加上第二个参数（一个数组 cache），这个数组来自组件实例，我们可以把内联事件处理函数添加到数组里。这样重新渲染时会优先读取缓存，避免了不必要的组件更新。



### 代码生成 generate

代码生成就是将 AST 变成渲染函数的代码字符串，所以本质上是字符串拼接的工作。通过递归 AST 来生成字符串，子节点字符串拼成后将其拼接到根节点字符串的参数中。我们只需提前为不同的 AST 节点（元素、文本、注释节点）编写对应的代码生成函数。匹配到哪一类 AST 节点，就调用哪一类的生成函数。



## patch 逻辑

Vue3 diff 借鉴了文本 diff 的预处理思路，先处理新旧节点的头尾相同部分，也就是前置节点和后置节点，缩小了乱序比对的区间。

如果还有剩下的，就建立节点的索引关系，即新节点在旧节点中的位置索引，然后构造出最长递增子序列，用于 DOM 移动操作，最长递增子序列所指向的节点即为不需要移动的节点，**减少了 DOM 的移动操作**，这就是相对于 Vue2 改进的地方。



## CSR SSR

### 服务端渲染

远古时代的网页主要强调内容本身，并不注重与用户的交互。当时主要采用服务端渲染。

服务端渲染流程：

- 用户通过浏览器向服务器发送请求
- 服务器从数据库拿到数据
- 服务器根据模板和获取的数据拼接出最终的 HTML 字符串并发送给浏览器
- 浏览器解析 HTML 内容并渲染

用户任何微小的操作都会重复上述步骤，导致页面刷新。

### 客户端渲染

后来 Ajax 兴起，催生 Web 2.0。这个阶段大量的单页面应用诞生，也就是客户端渲染。与 SSR 在服务端完成模板与数据的融合不同，CSR 在浏览器中完成这份工作（页面渲染是JS负责进行的）。

### CSR SSR 对比

- 服务端渲染 SEO 更友好，没有白屏问题
- 客户端渲染对服务端资源的消耗较少，没有真正的页面跳转用户体验更流畅



# 组件通信

父子 props，emit

attrs 用于批量传输数据

provide / inject 适合应用在插件中，实现跨级数据传递

$parent，$children，$ref 获取实例

eventBus 发布订阅，组件一多就会变得很混乱

Vuex



# 虚拟 DOM 理解

这里面有几个方面的问题

**1. 框架提供可维护性**

框架的意义在于为你掩盖底层的 DOM 操作，让你用声明式的方式来描述你的目的，从而让你的代码更容易维护。在构建应用的时候，你难道每个地方都手动更新 DOM 吗？出于可维护性，这是不可能的。框架给你的保证是，你在不需要手动优化的情况下，让你方便的维护，所以他必须要虚拟 DOM 这一中间层来实现。

**2. 性能**

任何框架都没有把虚拟 DOM 当作卖点。如果没有虚拟 DOM，就是直接重置 innerHTML。除了在大型列表里少量数据变化的情况下，重置 innerHTML 本就是合理的合理的操作。真正的问题是只有一行数据变了，它也需要重置全部，这时候显然就有大量的浪费。

我们可以比较一下 innerHTML vs Virtual DOM 的重绘性能消耗：

- innerHTML:  render html string + 重新创建所有 DOM 元素
- Virtual DOM: render Virtual DOM + diff + 必要的 DOM 更新

innerHTML 的总计算量和 DOM 总量相关， Virtual DOM 的计算量里面，js 计算和 DOM 总量相关，DOM 操作是和变动量相关。和 DOM 操作比起来，js 计算是极其便宜的。对于性能：它保证了不管什么场景，每次重绘的性能都可以接受。

**总结**

Virtual DOM 真正的价值从来都不是性能，对于性能它只保足下限。

意义：

- 提供声明式的维护方式，实现了 UI = f（data）也就是函数式 UI 编程
- 跨端，虚拟 DOM 是一层抽象，不仅仅可以抽象 DOM，其他场景也可以



# Vue3 Vue2 对比

monorepo的代码管理方式

Vue3 ts ->Vue2 flow

响应式 proxy ->defineProperty 一上来就递归重写 懒递归

模板编译优化

Vue3 diff算法（可以根据patchFlag 做diff）和vue2的区别(全量的diff) 最长递增子序列算法（并不是很重要，只是少了节点的移动操作）

options Api / compositionApi  逻辑横跳，利于 tree shaking



# VueX

Vuex 强依赖于 Vue，Vuex 的 state 会放到 Vue 的 data ，通过 Vue 的响应式系统收集使用它的组件，这就是核心。getter 就是代理到了 Vue 的 computed 上

单例模式，所有组件访问的都是同一个 store

发布订阅，把 mutation action 的函数都存起来

action 可以进行逻辑封装并复用，mutation 更改状态。严格模式下，内部用 watch 进行监听 state 变化，在 mutation 里修改 state 标志位就是 true，其他是 false，弹出警告。

收集并处理模块，没有namespace的时候getters都放在根上 ,actions ,mutations 会被合并数组 变成[a,b,c,d]容易处理

命名空间把 path 进行拼接 a/b/c 然后都加到名字前面，用来防止命名冲突



# VueRouter

递归树结构，转换为路由映射表，将 record 和 path 关联起来

但这还不够，a/b/c 不仅需要对应自己的组件，还需要 a 和 a/b 的组件，因为需要嵌套渲染

动态路由的实现就是将新路由插入到老的路由映射表里

hash -> hashChange   a 标签锚点，丑，服务端获取不到锚点内容（不能 SEO）

history -> popState 性能更高，但有兼容性问题，优先 history

路径变化需要重新渲染组件：把 current 变成响应式的，然后在 router-view 里使用 current，路径变化，current 变化，就可以重新渲染组件了

