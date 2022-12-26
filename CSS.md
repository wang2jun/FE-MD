# 选择器权重

在同一个元素使用不同的方式，声明了相同的css规则，**浏览器会通过权重来判断哪一种方式的声明，与这个元素最为相关，从而在该元素上应用css规则**。

伪类选择器：伪类（不存在的类，特殊的类）

伪类用来描述一个元素的特殊状态，比如：第一个子元素、被点击的元素、鼠标移入的元素.…

伪元素，表示页面中一些特殊的并不真实的存在的元素（特殊的位置）

- `::first-letter` 表示第一个字母

- `::first-line` 表示第一行

- `::selection` 表示选中的内容

- `::before` 元素的开始

- `::after` 元素的最后

- `::before`和`::after` 必须结合`content`属性来使用



## 等级关系

```markdown
!important > 行内样式 > ID选择器 > 类选择器、属性选择器、伪类选择器 > 元素选择器
```

## 要点

1.不推荐使用`!important`，因为`!important`根本没有结构与上下文可言，并且很多时候权重的问题，就是因为**不知道在哪里定义了一个`!important`而导致的。**

覆盖important:

虽然我们应该尽量避免使用!important，但你应该知道如何覆盖important，加点权重就可以实现

```css
    //!important 优先级最高，但也会被权重高的important所覆盖
    <button id="a" class="a">aaa</button>
    #a{
      background: blue  !important;  /* id的important覆盖class的important*/
    }
    .a{
      background: red  !important;
    }
```

2 选择器权重无法跨级比较

例子：无论多少个元素选择器，都没有一个class选择器权重高。



3 权重相同 后面为准

4 权重相同时，与元素距离近的选择器生效

```css
    #content h1 { // css样式表中
      padding: 5px;
    }
    <style type="text/css">
      #content h1 { 
          // html头部 因为html头部离元素更近一点，所以生效
        padding: 10px;
      }
    </style>
```

# link @import

1、link是html的标签，不仅可以加载 CSS 文件，还可以定义 RSS、rel 连接属性等；而@import是css的语法，只有导入样式表的作用。

2、DOM：javascript只能控制dom去改变link标签引入的样式，而@import的样式不是dom可以控制的。

3、link 权重高于 @import 的权重。

# 块级 行内元素

块级元素：一行放一个，可设置宽高，默认宽度为父元素宽度100%

div p h ul

行内元素：一行放多个，不可设置宽高，默认宽度为本身宽度 (**垂直方向**的 padding border margin 不影响布局)

span a button

行内块：一行放多个，可设置宽高，默认宽度为本身宽度

img input

# 外边距重叠 清除浮动 BFC

外边距重叠：相邻的两个盒子**（兄弟或者祖先）**的**垂直外边距**可以结合成一个单独的外边距。 

- 两个相邻的外面边距是正数时，折叠结果就是他们之中的较大值；
- 两个相邻的外边距都是负数时，折叠结果是两者绝对值的较大值；
- 两个外边距一正一负时，折叠结果是两者的相加的和；

BFC:是一个独立的渲染区域，**区域内与区域外部毫不相干**。

## BFC的布局约束规则

- BFC 内两个相邻 Box 的 margin 会发生重叠。
- BFC 中子元素不会超出他的包含块（包括浮动元素）。
- BFC 不会与浮动元素遮挡。

## BFC触发方式

float 不为 none

overflow 不为 visible

display 为 inline-block、table、flex、grid

position 值为 absolute、fixed

# 定位

| static   | 不开启定位，元素是静止的，默认值 |
| -------- | :------------------------------- |
| relative | 开启元素的相对定位               |
| absolute | 开启元素的绝对定位               |
| fixed    | 开启元素的固定定位               |
| sticky   | 开启元素的粘滞定位               |

定位元素垂直方向的位置由`top`和`bottom`两个属性控制，通常情况下只会使用其中之一

定位元素水平方向的位置由`left`和`right`两个属性控制，通常情况下只会使用其中之一

## 相对定位的特点

相对定位是参照于元素在文档流中的位置进行定位的（可以理解为相对于自身原始位置） 

相对定位会提升元素的层级（表现为可以覆盖其他元素）

相对定位不会改变元素的性质：块还是块，行内还是行内

相对定位没有脱离文档流

## 绝对定位的特点

开启绝对定位后，元素脱离文档流

改变元素的性质：行内变成块

使元素提升一个层级

相对于离当前元素最近的 开启了定位的祖先块元素进行定位的

## 固定定位的特点

固定定位也是一种绝对定位，所以固定定位的大部分特点都和绝对定位一样

不同：固定定位永远参照于浏览器的视口（viewport，可视窗口）进行定位，不会随网页的滚动条滚动

## 粘滞定位的特点

- 相对定位与固定定位的结合体

## 总结

| 定位方式             | 是否不设置偏移量，元素不会发生改变 | 是否脱离文档流 | 是否改变元素性质 | 是否提升元素层级 | 参考系                     |
| -------------------- | ---------------------------------- | -------------- | ---------------- | ---------------- | -------------------------- |
| relative（相对定位） | √                                  | ×              | ×                | √                | 参照于元素在文档流中的位置 |
| absolute（绝对定位） | ×                                  | √              | √                | √                | 参照于其包含块             |
| fixed（固定定位）    | ×                                  | √              | √                | √                | 参照于浏览器的视口         |
| sticky（粘滞定位）   | ×                                  | √              | √                | √                | 参照于浏览器的视口         |

# SEO

\1. 对网站的标题、关键字、描述精心设置，反映网站的定位，让搜索引擎明白网站是做什么的；

\2. 网站内容优化：内容与关键字的对应，增加关键字的密度；

\3. 在网站上合理设置Robots.txt文件；

\4. 生成针对搜索引擎友好的网站地图；

\5. 增加外部链接，到各个网站上宣传。

## 注意事项

### 结构布局优化：扁平化结构

（1）控制首页链接数量

（2）扁平化的目录层次

（3）导航优化：让爬虫能够清楚的了解网站结构，同时增加大量内部链接，方便抓取，降低跳出率。

（4）利用布局，把重要内容HTML代码放在最前：搜索引擎抓取HTML内容是从上到下

### 代码优化

1）突出重要内容---合理的设计 title、description和keywords

2）语义化书写HTML代码

3）a 标签：页内链接，要加 “title” 属性加以说明。而外部链接，则需要加上 rel="nofollow" 属性, 告诉不要去爬，因为爬虫爬了外部链接之后，就不会再回来了。

4）img 标签：加上 alt 属性

5）重要内容写在 HTML 里，爬虫爬不了 JS 输出的东西

6）少用 iframe ，爬虫一般不爬

# CSS 实现三角形

```css
    <style>
        div{
            position:absolute;
            border-top: 50px solid transparent;
            border-bottom: 50px solid blue;
            border-left: 50px solid yellow;
            border-right: 50px solid green;
        }
    </style>
```

# 垂直居中

```css
        /* 1 . 已知宽高  绝对定位 */ 
        div {
            position: absolute;
            top: 50%;
            left: 50%;
            margin-left: -100px;
            margin-top: -100px;
        } 

        /* 2 . 已知宽高  绝对定位  calc */
        /div {
            position: absolute;
            top: calc(50% - 100px);
            left: calc(50% - 150px);
        } 

        /* 3 . 未知宽高 */
        div {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
        } 

        /* 4 . flex */
        html {
            height: 100%;
            overflow: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
        }
```

# 固定宽高比

```css
        /* 1 . calc  */
        div {
            background-color: antiquewhite;
            width: 50vw;
            height: calc(50vw / 2);
        } 
        /* 2 padding-top  */
        div {
            background-color: antiquewhite;
            width: 50vw;
            height: 0;
            padding-top: calc(50vw * 1 / 2);
        }
```



# 移动端适配流程

- 百分比布局 ：基本不用，百分比相对于谁不确定

- rem + 动态的 html 的 font-size

  

  font-size 计算：

  媒体查询，动态更改不会进行更新

  JS 动态计算 font-size

  

  rem 单位换算：

  手动换算 / less、scss / postCss

- vw

不需要计算 font-size，不用担心被篡改

- flex



# Flex
