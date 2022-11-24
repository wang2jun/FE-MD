# let const

## let

### 块级作用域

let有块级作用域，var没有块级作用域。

在以前，只有全局作用域和函数作用域，没有块级作用域，这导致：大括号里面的变量可能覆盖外面的变量。

块级作用域的出现，使得（匿名函数 IIFE）没有必要了。

### 没有变量提升

目的：防止在变量声明前就使用这个变量

### 暂时性死区

```js
var tmp = 123;
if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```

暂时性死区：进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。

### 不允许重复声明

不允许在相同作用域内，重复声明同一个变量

## const

### 块级作用域

### 没有变量提升

### 暂时性死区

### 不允许重复声明

### 声明变量后不得改变

指的是变量指向的内存地址不得改动

### 声明时必须赋值

# 解构赋值

解构赋值本质属于“模式匹配”，只要等号两边的模式相同，左边的变量就会被赋予对应的值。

# 模板字符串

项目中会大量进行字符串拼接工作(尤其是**动态绑定数据**时)，传统模式字符串拼接是一个苦逼的任务。

拼接一套HTML字符串，还需要一行一行的处理，非常麻烦，而模板字符串只需要一行。

# Symbol

## 为何引入

ES5 的对象属性名都是字符串，这容易造成属性名的冲突。比如，你使用了一个他人提供的对象，但又想为这个对象添加新的方法，**新方法的名字就有可能与现有方法产生冲突**。如果有一种机制，保证每个属性的名字都是独一无二的就好了，这样就**从根本上防止属性名的冲突**。这就是 ES6 引入 Symbol 的原因。

**对象的属性名现在可以有两种类型，一种是原来就有的字符串，另一种就是新增的 Symbol 类型。**

```js
let s = Symbol();

typeof s // "symbol"
```

`Symbol() `函数前不能使用`new`命令，否则会报错。这是因为生成的 Symbol 是一个原始类型的值，不是对象，所以不能使用`new`命令来调用。**Symbol ，它是一种类似于字符串的数据类型。**

`Symbol()`函数可以接受一个字符串作为参数，表示对 Symbol 实例的描述。这**主要是为了在控制台显示，或者转为字符串时，比较容易区分。**

```js
let s1 = Symbol('foo');
let s2 = Symbol('bar');
s1 // Symbol(foo)
s2 // Symbol(bar)
s1.toString() // "Symbol(foo)"
s2.toString() // "Symbol(bar)"
```

## 应用

给类添加私有属性，元编程

## 内置的 Symbol 值

### Symbol.toStringTag

表示数据的类型，是个字符串，通常只有 Object.prototype.toString（） 方法才会去读它。

许多内置的 JavaScript 对象类型即便没有 `toStringTag` 属性，也能被 `toString()` 方法识别，有些是有的。而如果你自己创建类，就要自己加上这个属性才能被 Object.prototype.toString 读取。

# BigInt

## IEE754

计算机底层存储都是基于二进制的，需要事先由十进制转换为二进制存储与运算。

JavaScript 采用的是 IEEE 754 双精确度标准，无论是整数还是浮点数都用 64 个比特位（1+11+52）存。最高位是符号位, 指数位11位, 尾数位 52位。

IEEE754 还规定，有效数字第一位默认总是1 。因此，在尾数前面，还存在一个 “隐藏位” ，固定为 1，所以有效数字最长为 53 个。

所以 JavaScript 能表示的整数范围为：[-2^53，2^53]

对于浮点数，在转化为二进制时，尾数如果超过53位就会溢出，所以就要舍入，这会引起精度丢失。

进行浮点数运算时，要做**对阶、求和、舍入**，这个过程也会产生精度的丢失。

### EPSILON

```markdown
EPSILON 是 JavaScript 能够表示的最小精度。误差如果小于这个值，就可以认为已经没有意义了，即不存在误差了。EPSILON = 2^-52
```

```js
function epsilon(arg1, arg2){ 
    return Math.abs(arg1 - arg2) < Number.EPSILON;
} 
console.log(epsilon(0.1 + 0.2, 0.3)); // true
```

## 大整数

BigInt 只用来表示整数，没有位数的限制，多大的整数都可以表示。

为了与 Number 类型区别，BigInt 类型的数据必须添加后缀`n`。

```js
1234 // 普通整数
1234n // BigInt

1n + 2n // 3n
```

# 箭头函数

1.箭头函数语法上更简洁，简化了回调

2.箭头函数无 this， this 绑定定义该函数时的上下文（call/apply 等均无法改变）

3.箭头函数没有 arguments

4.箭头函数不能被 new（不能当构造函数），因为没有 this

长期以来，JS 的 this 一直是令人头痛的问题，在对象方法中使用 this ，必须非常小心。**箭头函数 ”绑定” this，很大程度上解决了这个困扰。**

# Iterator

Iterator 是一种接口，为不同的数据结构提供统一的访问机制。

数据只要具有 Symbol.iterator 属性，就可以认为是“可遍历的”（iterable）。原生具备 Iterator 接口的数据结构：String Array Set Map。

Object 之所以没有部署 Iterator 接口，是因为对象的哪个属性先遍历，哪个属性后遍历是不确定的，需要开发者手动指定。本质上，遍历器是一种线性处理，对于任何非线性的数据结构，部署遍历器接口，就等于部署一种线性转换。不过，严格地说，对象部署遍历器接口并不是很必要，因为这时对象实际上被当作 Map 结构使用，ES5 没有 Map 结构，而 ES6 原生提供了。

## 与其他遍历方式的比较

数组的 forEach：想要中途跳出 forEach 循环，只能主动抛错

对象的 for ... in ... :  会遍历到原型链上，遍历顺序不固定

# Set

Set 类似于数组，但是成员的值都是唯一的，没有重复的值。

Set 构造函数可以接受具有 iterable 接口的数据结构作为参数，用来初始化。

```js
const set = new Set([1, 2, 3, 4, 4]);
[...set] // [1, 2, 3, 4]

const items = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
items.size // 5
```

## 属性与方法

属性：

- `Set.prototype.size`：返回成员总数。

操作方法：

- `Set.prototype.add(value)`：添加某个值，返回 Set 结构本身。
- `Set.prototype.delete(value)`：删除某个值，返回一个布尔值，表示删除是否成功。
- `Set.prototype.has(value)`：返回一个布尔值，表示该值是否为`Set`的成员。
- `Set.prototype.clear()`：清除所有成员，没有返回值。

遍历方法：

- `Set.prototype.keys()`：返回键名的遍历器
- `Set.prototype.values()`：返回键值的遍历器
- `Set.prototype.entries()`：返回键值对的遍历器
- `Set.prototype.forEach()`：使用回调函数遍历每个成员

`keys`方法、`values`方法、`entries`方法**返回的都是遍历器对象**。由于 Set 结构没有键名，只有键值（或者说键名和键值是同一个值），所以`keys`方法和`values`方法的行为完全一致。

```js
let set = new Set(['red', 'green', 'blue']);

for (let item of set.keys()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.values()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.entries()) {
  console.log(item);
}
// ["red", "red"]
// ["green", "green"]
// ["blue", "blue"]
```

## 实现并集、交集、差集

```js
let a = new Set([1, 2, 3]);
let b = new Set([4, 3, 2]);

// 并集
let union = new Set([...a, ...b]);
// Set {1, 2, 3, 4}

// 交集
let intersect = new Set([...a].filter(x => b.has(x)));
// set {2, 3}

// （a 相对于 b 的）差集
let difference = new Set([...a].filter(x => !b.has(x)));
// Set {1}
```

## WeakSet

与 Set 不一样：

- WeakSet 的成员只能是引用型
- WeakSet 中的对象都是弱引用，即垃圾回收机制不考虑 WeakSet 对该对象的引用，因此 ES6 规定 WeakSet 不可遍历。

WeakSet 有以下三个方法。

- **WeakSet.prototype.add(value)**：向 WeakSet 实例添加一个新成员。
- **WeakSet.prototype.delete(value)**：清除 WeakSet 实例的指定成员。
- **WeakSet.prototype.has(value)**：返回一个布尔值，表示某个值是否在 WeakSet 实例之中。

# Map

## 为何引入

JavaScript 的对象（Object），本质上是键值对的集合（Hash 结构），但是传统上只能用字符串当作键。这给它的使用带来了很大的限制。Map 类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。**Object 结构提供了“字符串—值”的对应，Map 结构提供了“值—值”的对应，是一种更完善的 Hash 结构实现。如果你需要“键值对”的数据结构，Map 比 Object 更合适。**

任何具有 Iterator 接口、且每个成员都是一个双元素的数组的数据结构都可以当作`Map`构造函数的参数。

```js
const set = new Set([
  ['foo', 1],
  ['bar', 2]
]);
const m1 = new Map(set);
m1.get('foo') // 1

const m2 = new Map([['baz', 3]]);
const m3 = new Map(m2);
m3.get('baz') // 3
```

## 属性和方法

size 属性

操作方法：get set delete has clear

遍历方法：keys values entries forEach

keys values entries 返回的是遍历器

## WeakMap

`WeakMap` 与 `Map`的区别：

- `WeakMap` 只接受引用型数据作为键名
- `WeakMap`的键名所指向的对象，不计入垃圾回收机制（弱引用），所以不可遍历

### 场景

WeakMap 的专用场合就是，它的键所对应的对象，可能会在将来消失。WeakMap 结构有助于防止内存泄漏。

典型场合就是 DOM 节点作为键名，这个 DOM 节点删除，该键也会消失。

# Reflect

## 设计目的

（1） 将`Object`对象的一些**明显属于语言内部的方法**（比如 Object.defineProperty），放到`Reflect`对象上。现阶段，某些方法同时在Object和Reflect对象上部署，未来的新方法将只部署在`Reflect`对象上。也就是说，**从`Reflect`对象上可以拿到语言内部的方法**。

（2） **修改**某些 `Object` 方法的**返回结果**，**让其变得更合理**。比如，Object.defineProperty(obj, name, desc) 在无法定义属性时，会抛出一个错误，而 Reflect.defineProperty(obj, name, desc)则会返回`false`。

（3） 命令式 - > 函数式，比如 `name in obj` 和 `delete obj[name]` ，而Reflect.has(obj, name) 和 Reflect.deleteProperty(obj, name)  让它们变成了函数行为。

（4）`Reflect`对象的方法与`Proxy`对象的方法一一对应，只要是`Proxy`对象的方法，就能在`Reflect`对象上找到对应的方法。这就让`Proxy`对象可以方便地调用对应的`Reflect`方法，完成默认行为。也就是说，不管`Proxy`怎么修改默认行为，你总可以在`Reflect`上获取默认行为。

# Proxy

用于修改某些操作的默认行为，等同于在语言层面做出修改，所以属于元编程，即对编程语言进行编程。

Proxy 在目标对象之前加一层代理，外界对该对象的访问，都必须先通过这层，因此可以对外界的访问进行改写。

## this 问题

在 Proxy 代理的情况下，目标对象内部的 `this` 会指向 Proxy 。

# Class

## 由来

生成实例对象的传统方法是通过构造函数，这和主流的面向对象语言（ C++ 和 Java）差异很大，很容易让新学习这门语言的程序员感到困惑，**因此提供了更接近面向对象语言的写法**。

```js
class Animal {
    constructor() {
        this.type = '哺乳类'
    }
    //原型上的属性
    get b() {
        return 2
    }
    static flag = '动物' //静态属性
}
// es6中的继承
// Tiger.__proto__ = Animal 父类的静态属性和静态方法通过浅拷贝继承
// Animal.call(this)  实例
// Tiger.prototype.__proto__ = Animal.prototype 原型
class Tiger extends Animal {
    constructor() { 
        // 子类构造函数在使用 this 之前必须调用 super
        // 先生成一个继承父类的对象，再将该对象作为子类的实例
        super() //  Animal.call(this);
        console.log(super.b) // super 指向父类的原型
    }
    static getFlag() {
        return super.flag  //静态方法中的super指向的是父类
    }
    eat() {
        super.eat() // super指向父类的原型
    }
}
```

# Promise

异步:

- 并发（使用for循环迭代执行) 
- 串行（借助回调第一个完成后完成第二个）

## 优势

- 将异步操作以链式调用的流程表达出来，避免了层层嵌套的回调。

- 多个请求并发（希望同步最终的结果）Promise.all

## 特点

以状态为核心且状态不受外界影响。有三种状态：`pending`（进行中）、`fulfilled`（已成功）和`rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是 Promise 这个名字的由来，它的英语意思就是承诺，表示其他手段无法改变。并且一旦状态改变，就不会再变。

## 缺陷

- **Promise 无法被中断**  ，一旦新建它就会立即执行。想要中断（其实就是忽略结果），就得让 promise 一直保持在 pending，执行不了成功或失败的回调。
- 当处于 pending 状态时，无法得知目前到哪一个阶段（刚刚开始还是即将完成）
- 如果不设置回调捕获错误，Promise 内部的错误，不会反应到外部，也就是不会中断整个脚本的执行。
- 另外，**Promise 依旧基于回调 ！**原来的任务被 Promise 包装了一下，不管什么操作，一眼看去都是一堆 then ，原来的语义变得很不清楚。

## Promise.prototype.catch()

```js
Promise.prototype.catch = function (errCallback) {
    return this.then(null, errCallback)
}
```

推荐 .then.catch 而不是 .then(resolve,reject)，因为 catch 可以捕获前面 then 方法执行中的错误，也更接近同步的写法。

## Promise.prototype.finally()

不管 Promise 最后状态如何，都会执行这个方法。如果不用 finally ，用 then 需要为成功和失败两种情况各写一次。有了 finally 方法，则只需要写一次。

```javascript
Promise.prototype.finally = function (callback) {
    return this.then((value) => {
        return Promise.resolve(callback()).then(() => value)
    }, (reason) => {
        return Promise.resolve(callback()).then(() => { throw reason })
    })
}
```

## Promise.all()

Promise.all 用于实现并发

```js
const isPromise = value => typeof value.then === 'function';
Promise.all = function (promises) { //全部成功才成功
    return new Promise((resolve, reject) => {
        //异步: 并发（使用for循环迭代执行) 和串行（借助回调第一个完成后完成第二个）
        let arr = []; //遍历数组  依次拿到执行结果
        let index = 0;
        const processData = (key, data) => {
            arr[key] = data;//不能使用数组的长度来计算
            if (++index === promises.length) {
                resolve(arr);
            }
        }
        for (let i = 0; i < promises.length; i++) {
            let result = promises[i];
            if (isPromise(result)) {
                result.then((data) => {
                    processData(i, data)
                }, reject)
            } else {
                processData(i, result)
            }
        }
    });
}
```

## Promise.race()

```js
const isPromise = value => typeof value.then === 'function ';
Promise.race = function (promises) {
    return new Promise((resolve, reject) => {
        for (let i = 0; i < promises.length; i++) {
            let result = promises[i];
            if (isPromise(result)) {
                result.then(resolve, reject)
            } else {
                resolve(result);
            }
        }
    });
}
```

### 应用：请求超时

```js
let promise = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve('ok')
    }, 3000)
})
const wrap = promise => {
    let abort;
    let myP = new Promise((resolve, reject) => {
        abort = reject;
    })
    let p = Promise.race([promise, myP]);
    p.abort = abort;
    return p;
}
let p = wrap(promise);
p.then(data => {
    console.log(data);
}, (err) => {
    console.log(err)
})
setTimeout(() => {
    p.abort('promise 超时');
}, 2000)
```

## Promise.resolve()

```javascript
Promise.resolve = function (data) {
    return new Promise((resolve, reject) => {
        resolve(data);
    })
}
```

如果参数是 Promise 实例，那么将原封不动地返回这个实例。

如果参数不是 Promise 实例，或者不传，则返回一个新的状态为成功的 Promise 。

## Promise.reject()

```js
Promise.reject = function (reason) {
    return new Promise((resolve, reject) => {
        reject(reason);
    })
}
```

Promise.reject() 的参数，会原封不动地作为 reject 的理由传递下去。

# Generator

## 特点

- 普通函数默认会从头到尾执行，而 generator 执行过程中碰到 yield 可以暂停执行。
- 执行 Generator 函数会返回一个遍历器对象。并且它的 next 很特殊，除了第一次之外的 next 方法，其他都是把 next 中的参数传递给上一次 yield 的返回结果。

## 原理

Generator 执行产生的上下文，一旦遇到 yield 命令，就会暂时退出堆栈，但是并不消失，里面的所有变量和对象会冻结在当前状态。等到对它执行 next 命令时，这个上下文环境又会重新加入调用栈，冻结的变量和对象恢复执行。

把函数分段，再弄个指针，执行的时候 while 循环，每次都改变偏移量移动指针（命中 switch case），一段一段执行。

## 异步应用

Generator 函数的暂停执行的效果，**意味着可以把异步操作写在 yield 表达式里面，等到调用 next 方法时再往后执行**。这实际上**等同于不需要写回调函数**了。所以，Generator 函数的一个重要实际意义就是用来**处理异步操作，改写回调函数，代码编写更像是同步**。

## 缺陷

流程管理很不方便，需要手动执行

## CO 库

TJ CO 库：自动执行 Generator

Generator 就是一个异步操作的容器。它的自动执行需要一种机制，当异步操作有了结果，能够继续执行。CO 将异步操作包装成 Promise 对象，用 then 方法继续执行。

```js
const co = it => {
    return new Promise((resolve, reject) => {
        // 串行靠的是回调
        function next(data) {
            let { value, done } = it.next(data);
            if (!done) {
                Promise.resolve(value).then(next, reject); // 成功了就递归
            } else {
                resolve(value);
            }
        }
        next();
    });
}
```

# async await

async + await = generator + co

用 async 替换 *，用 await 替换 yield。默认 async 函数返回 promise

## 优化 Generator

- 自执行
- 更好的语义

## 错误处理

async 本身就是多个异步操作包装成的一个 Promise 对象。所以 async 函数内部抛出的错误，会导致返回的 Promise 对象变为 reject 状态，我们就可以**在外面用 catch 接收**。并且如果某个 await 后面的 promise 失败了就代表了整个 promise 失败了，async 被中断了，错误也会被外面的 catch 接收。

我们希望某个异步操作失败，也不要中断后面的异步操作。这时可以**将 await 放在 try...catch 结构里面**，这样不管这个异步操作是否成功，后面的 await 都会执行。另一种方法是 **await 后面的 Promise 再跟一个 catch 方法来捕获错误**。

## 并发

```js
let foo = await getFoo();
let bar = await getBar(); // 串行

// 并发的两种写法
// 写法一 Promise.all
let [foo, bar] = await Promise.all([getFoo(), getBar()]);
// 写法二 迭代
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```

# 模块化

## 初期

一开始写 js，通过 script 标签把 js 文件引入，一个html页面对应一个js文件。如果出现公用的 js 逻辑，我们提取到公用的 js 文件，需要的html页面就引入。js逻辑拆分的越来越细，一个 html 页面需要引入的 js 文件也越来越多。这就导致了问题：**js文件如果存在依赖关系，我们就得注意引入顺序；同时多个js文件共用一个全局作用域，很容易命名冲突。**

IIFE ：函数声明被括号括起来了它就变成了表达式，就可以执行。把数据声明到立即执行函数中，就解决了污染全局的问题。公用的数据就 return 出去（其实就是闭包），其他模块要用的话就传参进去。这样IIFE就实现了模块之间独立且不污染全局还能相互依赖，但是加载顺序没有解决。 

## CommonJS

到此为止，js 模块化都依赖于html页面。node.js 搞出了模块化，CommonJS 规范也应运而生。

用 require 引入时就会创建一个模块实例，并且有缓存，多次 require 只执行一次。

但是 common.js 只能在服务端运行，所以出现了能在客户端运行的 AMD 规范。

## AMD （异步模块定义）

最佳实现 ：require.js

依赖前置：依赖的模块加载完才执行当前模块的回调，并且加载是异步的（利用了  script 的 async 标签），所以 AMD 不在意模块引入顺序

## CMD （通用模块定义）

实现 ：sea.js

与 AMD 最大的区别：依赖就近，按需加载，用到哪个模块就加载哪个模块，这是个效率问题的改进

## ES Module

分别暴露 统一暴露 默认暴露

es5靠的语法取巧，AMD CMD 都是民间社区提供的方式，ECMA 终于干了点事。ES3 到ES5 更新的很少，ECMA 顶不住压力，被骂得太惨，才有了大变更 ES6（ js 设计之初，也没想到 js 发展的这么快，ECMA 跟不上脚步了）

commonjs 运行时，输出的是值的浅拷贝

es6 模块化：模块的依赖关系，导入导出的变量编译时就已经决定了（设计思想是尽量的静态化），输出的是值的引用

## 循环加载

CommonJS 模块的重要特性是加载时执行，即脚本代码在 require 的时候，就会全部执行。一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。

ES6 模块是动态引用，使用 import 从一个模块加载变量，那些变量不会被缓存，我们会拿到一个指向被加载模块的引用，需要开发者自己保证，真正取值的时候能够取到值，也就是说你暴露的那个接口是有效的。
