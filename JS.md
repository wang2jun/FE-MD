# ASCII、Unicode、UTF-8

计算机通过二进制存储数据，所以我们需要一份对照表。每个字符都有对应的数字，叫做码点。ASCll 的码点为 0~127，码点与对应字符的集合叫字符集，显然 ASCll 字符集不能全世界通用。于是有了通用的 Unicode 字符集。

但是字符不一定会以对应码点存储在计算机里，编码方式才决定字符到计算机存储内容的映射。UTF-32 直接存储码点，不够32位，就在前面补零（不然计算机分不清字符从哪到哪），但也导致空间浪费。UTF-8 是针对 Unicode 的可变长度编码，针对不同的码点范围，存储时规定字节的前缀，保证了字符分割能让计算机认识还能合理使用空间。

# 严格模式

有更多的限制：

- 变量必须声明后再使用，防止意外创建全局变量
- 禁止 this 指向全局对象
- 不能使用 with



# 类型转换规则

类型转换只有三种：

- 转换成数字
- 转换成布尔值
- 转换成字符串

转 bool：

- 对于数字，除了+0、-0、NaN 都为 true
- 字符串除了空串，都为 true
- 引用型转bool都是true

转字符串：

- 数字直接转
- bool、函数、symbol转为 ”true“
- 对象转成 "[object Object]"

转数字：

- 字符串转数字，如果字符串里是数字直接转，否则NaN
- 数组：空数组为0，一个元素且为数字的话转数字，其他情况NaN
- null 0，undefined NaN
- 引用型 NaN



== 类型转换规则：

- 两边的类型是否相同
- 是否是null和undefined，是的话就返回true
- 判断类型是否是String和Number，是的话，把String 类型转换成 Number，再进行比较
- 判断其中一方是否是Boolean，是的话就把Boolean转换成Number，再进行比较
- 如果其中一方为Object，且另一方为String、Number，会将Object转换成字符串，再进行比较



对象转原始类型：

1. 如果Symbol.toPrimitive()方法，优先调用再返回
2. 调用valueOf()，如果转换为原始类型，则返回
3. 调用toString()，如果转换为原始类型，则返回
4. 如果都没有返回原始类型，会报错



# 原始值包装类型

为了方便操作原始值，提供了特殊的引用类型：Boolean、Number 、 String。 每当用到某个原始值的方法时，引擎都会创建一个相应原始值包装类型的对象，从而暴露出操作原始值的各种方法。

(1) 创建一个 String 类型的实例

(2) 调用实例上的特定方法

(3) 销毁实例



# 数据类型检测

## typeof

原理：基于数据的二进制

null 检测结果为  object ：对象以000开头，null为全0，所以结果是object

检测引用类型时，只能准确检测 function，其余都显示 object

## instance of

原理 : A instance of  B  ，看 B 的 prototype 是否在 A 的原型链上

不能检测基本数据类型，可以检测引用类型

由于我们可以修改原型指向，所以这个方法并不安全

## constructor

可以检测除 null 和 undefined 以外的基本类型以及引用类型

可以修改 constructor 指向，所以不安全

## Object.prototype.toString.call()

找 Symbol.toStringTag



# this

**this 指的是把函数当成方法调用的上下文对象**，也就是说谁调用函数谁就是 this

函数执行阶段：方法前的"点"，如果没有，就是window（严格模式是 undefined ）

构造函数中 this 是当前类的实例

给 DOM 绑定事件，方法中的 this 是当前 DOM

**箭头函数的 this 绑定定义该函数时的上下文**

# new

```js
function mockNew(Cons, ...params) {
    let obj = {};
    obj.__proto__ = Cons.prototype;
    Cons.call(obj, ...params);
    return obj;
}
function Animai(name, age) {
    this.name = name;
    this.age = age
}
console.log(mockNew(Animai, 'wj', 123))
```

# call apply bind

call apply 区别是参数（一个一个传和数组传）

bind 不立即执行,返回一个函数

```js
// call 原理：obj.fn()   fn的this是.前面的对象
Function.prototype.call = function (context, ...params) {
    let fn = this
    let key = Symbol()
    context[key] = fn
    context[key](...params)
    delete context[key]
}
// bind
Function.prototype.bind = function (context, ...params) {
    let fn = this
    return function () {
        fn.call(context, ...params)
    }
}
```



# 执行上下文 作用域

执行上下文即是代码的执行环境，分为全局上下文和函数上下文。每个上下文都有一个关联的变量对象，对于函数则称为活动对象，在上下文中定义的所有变量都存在于这个对象上。全局上下文是最外层的上下文，也就是宿主环境，在浏览器中是 window。上下文在其所有代码都执行完毕后会被销毁，包括定义在它上面的所有变量，全局上下文直到关闭网页才会销毁。

执行流进入函数时，函数上下文被压入上下文栈。 在函数执行完之后，上下文栈弹出该函数上下文，将控制权返还给之前的执行上下文。程序的执行流就是通过上下文栈进行控制的。

创建上下文时，也会创建变量对象的作用域链。作用域链决定访问变量时的顺序。正在执行的上下文的变量对象始终位于作用域链的最前端。作用域链中的下一个变量对象来自上一层上下文，全局上下文的变量对象始终是作用域链的最后一个变量对象。



# DOM 事件流

事件流分为 3 个阶段：捕获、到达目标、冒泡。

addEventListener（）接收 3 个参数：事件名、事件处理函数、布尔值，true 表示在捕获阶段事件触发，false（默认值）表示在冒泡阶段触发。

事件触发一般在冒泡阶段，主要原因是浏览器兼容性好。捕获阶段触发通常用于在事件到达目标之前拦截事件。如果不需要拦截，则不要使用捕获。冒泡阶段触发一般应用于事件委托。

在 event 对象中，this、currentTarget 等于绑定事件的 DOM，target 等于事件的实际目标（比如你点的元素是谁，target 就是谁）



# xhr fetch

```js
let xhr = new XMLHttpRequest();
xhr.onreadystatechange = function () {
    if (xhr.readyState == 4) {
        if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304) {
            alert(xhr.responseText);
        } else {
            alert("Request was unsuccessful: " + xhr.status);
        }
    }
};
xhr.open("get", "example.txt", true);
xhr.timeout = 1000; // 设置 1 秒超时
xhr.ontimeout = function () {
    alert("Request did not return in a second.");
};
xhr.send(null);
// 如果想取消异步请求，可以调用 abort()方法：
xhr.abort();
```

```js
fetch('bar.txt')
    .then((response) => {
        console.log(response);
    }); 
```

fetch 能在 service worker 中使用，基于 Promise，更容易使用，是更现代化的 API。并且是对 HTTP 接口的抽象，设计的更加底层，有好的扩展性。

xhr 的写法比较繁琐混乱

- fetch 只对网络请求报错，服务器返回 400，500 错误码时并不会 reject，只有网络错误这些导致请求不能完成时，fetch 才会被 reject。
- fetch 默认不会带cookie，要自己添加添加配置
- fetch 的 abort 还在实验阶段
- fetch 不支持超时控制，也不能监测请求的进度，而 xhr 可以。



fetch 终会取代 xhr，但过程比较漫长。

# 遍历对象

for … in

- 遍历可枚举属性（不能遍历 symbol ）
- 基于原型链一级一级查找，性能消耗极大
- 遍历顺序：非负整数升序遍历，其他属性按照创建时候的顺序遍历

```js
Object.keys()
// 遍历可枚举属性

Object.getOwnPropertySymbols() // Symbol
Object.getOwnPropertyNames() // 可枚举+不可枚举 不包括Symbol

Reflect.ownKeys(target) = Object.getOwnPropertyNames(target) + Object.getOwnPropertySymbols(target);

```

# 函数柯里化

```javascript
const currying = (fn, arr = []) => {
    let len = fn.length; // 这里获取的是函数的参数的个数
    return function (...args) {
        let concatValue = [...arr, ...args];
        if (concatValue.length < len) {
            return currying(fn, concatValue); // 递归不停的产生函数
        } else {
            return fn(...concatValue);
        }
    }
}
```

# 发布订阅

发布（emit）订阅（on）

订阅 on 就是把一些函数维护到一个数组中

发布 emit 就是让数组中的方法依次执行

特点：订阅和发布没有明显的关联， 靠中介（数组）来做事

```js
class Event {
  constructor() {
    this.arr = [];
  }
  on(fn) {
    this.arr.push(fn);
  }
  emit() {
    this.arr.forEach((fn) => fn());
  }
}
let e = new Event();
e.on(() => console.log("on"));
e.emit();
```

# 观察者模式

观察者模式基于发布订阅模式

有观察者 被观察者。被观察者收集观察者，被观察者状态发生变化要主动通知观察者

```js
class Subject { // 被观察者  小宝宝
    constructor(name) {
        this.name = name;
        this.state = '开心的';
        this.observers = [];
    }
    attach(o) {
        this.observers.push(o);
    }
    setState(newState) {
        this.state = newState;
        this.observers.forEach(o => o.update(this))
    }
}
class Observer { // 观察者  我  我媳妇
    constructor(name) {
        this.name = name
    }
    update(baby) {
        console.log('当前' + this.name + '被通知了', '当前小宝宝的状态是' + baby.state)
    }
}
let baby = new Subject('小宝宝');
let parent = new Observer('爸爸');
let mother = new Observer('妈妈');
baby.attach(parent);
baby.attach(mother);
baby.setState('被欺负了');
```

# 深拷贝

JSON.stringify：

JSON 只支持：字符串、数字、布尔、 null，对象、数组



```js
function deepClone(obj, hash = new WeakMap()) {
  // 弱引用，不要用 map
  if (obj == null) return obj; // null 和 undefiend
  if (typeof obj !== "object") return obj;
  if (obj instanceof RegExp) return new RegExp(obj);
  if (obj instanceof Date) return new Date(obj);
  if (hash.get(obj)) return hash.get(obj);
  let cloneObj = new obj.constructor();
  hash.set(obj, cloneObj);
  Reflect.ownKeys(obj).forEach((key) => {
    cloneObj[key] = deepClone(obj[key], hash);
  });
  return cloneObj;
}
```

# 事件循环

JavaScript 用途是与用户互动和操作 DOM。这决定了它只能是单线程，否则会带来很复杂的同步问题。

单线程导致一个任务执行需要等前面的任务都执行完，所以就需要解决单个任务占用主线程过久的问题。所以我们要划分同步任务和异步任务。

与此同时大部分任务都是宏任务，各种 IO 事件、JS 同步任务、用户交互随时都有可能被执行，这样我们无法控制某个任务的执行时机，对于时间精度要求较高的需求就没办法满足了。所以我们又引入了微任务，执行完当前宏任务之后立刻执行所有微任务。



宏任务包括： script 代码块、setTimeout、setInterval、I/O

微任务包括： promise.then 、MutationObserver



一次循环：一个宏任务，清空微任务队列，UI 渲染

事件循环过程：

1. 执行 script 代码块，碰到宏任务就交给定时器监听线程，定时器监听线程发现时间到了就把它添加到宏任务队列里，碰到微任务就添加到微任务队列里。
2. 执行栈任务执行完，就去清空微任务队列。如果执行微任务过程中又遇到微任务，就继续添加到微任务队列末尾继续执行，把微任务全部执行完。
3. UI 渲染
4. 检查宏任务队列是否为空，有就取出队头的任务压入执行栈中执行
5. 执行宏任务，无限循环



# 防抖

```js
// 场景:搜索框
function debounce(fn, delay) {
    let timer
    return function (...arg) {
        clearTimeout(timer)
        timer = setTimeout(() => {
            fn.call(this, ...arg)
        }, delay)
    }
}
```



# 节流

```js
// 场景：提交表单，规定时间内只能提交一次，时间过了才能再提交
function throttle(fun, time) {
    let t1 = 0 //初始时间
    return function (...arg) {
        let t2 = new Date() //当前时间
        if (t2 - t1 > time) {
            fun.call(this, ...arg)
            t1 = t2
        }
    }
}
```



# 基于对象

**JavaScript 不是一门面向对象的语言**，**因为面向对象语言天生支持封装、继承、多态**。

JS 对封装、多态支持的不好，并且实现继承的方式和面向对象的语言存在很大的差异。

面向对象语言对继承做了充分的支持，提供了大量的关键字，如 public、 protected 等，使得面向对象语言的继承变得异常繁琐和复杂，而 JavaScript 是**基于原型链继承**。



# 函数是一等公民

JavaScript 中的**函数就是一种特殊的对象**。可以将函数赋值给一个变量，函数可以作为参数，函数可以返回函数。使得代码逻辑清晰，简洁，而且很容易实现一些特性，比如**闭包、函数式编程**。

所以函数是一种特殊的对象，不同的是，函数可以被调用。

## 为什么函数可以被调用？

函数对象有两个特殊属性：name 、 code 。

name属性的值就是函数名称，如果函数没有设置函数名，name 属性值就是 anonymous （匿名）。

code 属性值是函数代码，以字符串的形式存储。当执行到一个函数调用语句时，引擎便会从函数中取出 code 值，然后执行。



# 垃圾回收

- 标记清除
- 引用计数

标记清除：垃圾回收程序运行的时候，会标记内存中存储的所有变量。然后将当前上下文中的变量，以及被上下文的变量引用的变量的标记去掉。在此之后有标记的变量就是待删除的了，原因是任何在上下文中的变量都访问不到它们了。随后做内存清理收回内存。（详见 V8 垃圾回收）

引用计数：思路是对每个值都记录它被引用的次数。一个值被赋给另一个变量，那么引用数加 1。类似地，引用数减 1。当值的引用数为 0 时，就说明没办法再访问到这个值，因此可以收回其内存了。但是如果循环引用，会导致对象永远无法被销毁。



# 闭包

闭包指引用了另一个函数作用域中变量的函数（通常在嵌套函数中出现）

在定义函数时，就会为它创建作用域链，预装载变量对象，并保存在内部的[[Scope]]中。在调用这个函数时，会创建相应的执行上下文，然后通过复制函数的[[Scope]]来创建其作用域链。接着会创建函数的活动对象，并推入作用域链的前端。作用域链其实是一个包含指针的列表，每个指针分别指向一个变量对象。函数执行完毕后，函数的活动对象会被销毁。

而闭包则是在一个函数内部定义的函数会把外部函数的活动对象添加到自己的作用域链中。正因为内部函数的作用域链中仍然有对外部函数活动对象的引用，活动对象并不能在它执行完毕后销毁。外部函数上下文会销毁，但它的活动对象仍然会保留在内存中，直到闭包被销毁后才会被销毁。

## 应用

- 实现方法和属性的私有化，不受外部干扰
- 最典型的应用场景：IIFE，模拟块级作用域实现模块化
- 柯里化、防抖、节流
- 闭包无处不在，所有的封装都是基于闭包，实质是高阶函数，函数是一等公民

引擎对闭包的实现：惰性解析，预解析器



# 几种内存问题

## 内存泄露

定义：当进程不再需要某些内存的时候，这些内存依然没有被回收。

原因：

1.意外的全局变量

没有声明直接使用变量（用严格模式避免）

2.没有清除的计时器

清除定时器之前，回调函数里的变量以及回调函数本身以及变量都无法被回收。

3.不正当使用闭包

只需要返回 x，却返回了 obj.x

使用完返回的闭包变量，要置空

4.没有清理的DOM元素引用

### 在浏览器检测内存泄漏

[掘金连接](https://juejin.cn/post/6984188410659340324)

开发者工具 Memory 选项  录制多个快照

查看New、Deleted、Delta三项，对比内存占用

## 内存膨胀

内存膨胀主要表现在程序员对内存管理的不科学，比如只要 50M 内存就可以搞定的，有些程序员却花费了 500M 内存。

通常表现为内存在某一段时间内快速增长，然后达到一个平稳的峰值继续运行。

比如一次性加载了大量的资源，内存会快速达到一个峰值。

## 频繁的垃圾回收

频繁使用大的临时变量，导致了新生代空间很快被装满，从而频繁触发垃圾回收。频繁的垃圾回收操作会让你感觉到页面卡顿。

如何解决：可以考虑将这些临时变量设置为全局变量。



# requestAnimationFrame

用来替代曾经的 setInterval 、setTimeout 函数，用于动画绘制，浏览器在下次重绘之前执行回调更新动画，以浏览器的显示频率来作为其动画的频率（所以不会造成掉帧）

当运行在后台标签页时，requestAnimationFrame 会被暂停调用，提升性能

执行时机：**宏任务 微任务 requestAnimationFrame UI 渲染**

## 对比

因为 setTimeout 和 setInterval 是异步宏任务，必须等同步任务、微任务执行，才会去执行回调。即使给定毫秒数 ，也无法把控执行时机。

最大的优势 ：由浏览器来决定回调函数的执行时机

## 优势

- 动画效果更好
- 性能更好



# 图片懒加载

```js
const images = document.querySelectorAll('img');
window.addEventListener('scroll', (e) => {
    images.forEach(image => {
        const imageTop = image.getBoundingClientRect().top;
        if (imageTop < window.innerHeight) {
            const data_src = image.getAttribute('data-src');
            image.setAttribute('src', data_src);
        };
        console.log('scroll触发');
    });
});
```

缺陷：

1. 需要设置 isLoad 属性，使得图片加载完成后不再重复触发逻辑

2. 手动节流

```js
const images = document.querySelectorAll("img");
const callback = entries => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            const image = entry.target;
            const data_src = image.getAttribute("data-src")
            image.setAttribute("src", data_src)
            observer.unobserve(image)
            console.log("触发")
        }
    });
};
const observer = new IntersectionObserver(callback);
images.forEach(image => {
    observer.observe(image)
})
```



# DOMContentLoaded 与 load

- 当 DOMContentLoaded 事件触发时，仅 DOM 加载完成，不包括样式表，图片。

- 当 onload事件触发时，DOM，样式表，JS 脚本，图片都已经加载完成了



# defer async

defer：要等到 DOM 树生成，其他 JS 脚本执行完，才会执行。

async：下载完就会执行



# WebWorker

WebWorker 是一个子线程，完全受主线程控制，不能操作DOM（ JS 单线程仍然未改变）。

WebWorker 可以理解为浏览器给主线程开的外挂，专门用来解决那些大量计算问题。如果有非常耗时的工作，单独开一个 WebWorker 线程，等到计算出结果交给主线程即可。

## SharedWorker

- WebWorker 只属于某个页面（渲染进程）
- 浏览器单独开辟进程管理 SharedWorker 运行 JS



# cookie/Storage/IndexedDB

Cookie 用来解决 HTTP 无状态的问题，向同一个域名下发送请求，都会携带相同的 Cookie。

缺陷：

- 容量只有 4KB，只能存储少量信息
- 性能缺陷。不管域名下面的某一个地址需不需要 Cookie ，请求都会带上完整的 Cookie，带来性能浪费。
- 安全缺陷，JS 和发请求时都能够获取到 cookie，所以有了 HttpOnly 防范 JS 读取（XSS），SameSite 防范第三方请求携带 cookie（CSRF）

localStorage 和 cookie 相同点是针对同一域名存储。容量为 5M，只存在于客户端（不参与服务端通信）。我们一般利用 localStorage 的较大容量和持久性，存储一些稳定的资源，如官网的 logo，Base64 格式的图片、搜索记录。

sessionStorage 和 localStorage 唯一的区别是前者是会话级别的存储，并不是持久化存储。页面关闭，sessionStorage 就不存在了。可以用来对表单信息进行维护，将表单信息存储在 sessionStorage，保证页面刷新也不会让之前的表单信息丢失。如果关闭页面后不需要历史记录，也可以用来存储本次浏览记录。

IndexedDB 是运行在浏览器中的数据库，所以理论上没有容量上限。 除了拥有数据库本身的特性，IndexedDB 也受同源策略限制，无法访问跨域的数据库。

**总结**

1. cookie 不适合存储，存在非常多的缺陷。
2. localStorage、sessionStorage, 默认不会参与和服务器的通信。
3. IndexedDB 为大型数据的存储提供了接口。



# Tree 扁平化

```js
// 扁平转 tree
function dataToTree(data) {
  function parseChildren(parent, children) {
    parent.forEach((p) => {
      children.forEach((c, i) => {
        if (c.pid === p.id) {
          if (p.children) p.children.push(c);
          else p.children = [c];
          let _children = JSON.parse(JSON.stringify(children));
          _children.splice(i, 1);
          parseChildren([c], _children);
        }
      });
    });
  }

  let parents = data.filter((val) => !val.pid),
    children = data.filter((val) => val.pid);
  parseChildren(parents, children);
  return parents;
}
```

```js
// tree 扁平化
function treeToArray(resultArr, children) {
  children.forEach((obj) => {
    if (obj.children) {
      treeToArray(resultArr, obj.children);
      delete obj.children;
      resultArr.push(obj);
    } else {
      resultArr.push(obj);
    }
  });
}
let res = [];
treeToArray(res, tree);
```

