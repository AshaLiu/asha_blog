---
title: CSS之流、元素、基本尺寸
date: 2019-06-02 17:54:16
tags: css
---

<a name="04f9d6ef"></a>
### css的历史环境
css的全称是 `Cascading Style Sheets`，翻译成中文是 `层叠样式表`。所谓 `层叠`，意思是样式可以层层叠加，比如说，一个页面的元素都继承了12像素的大小，某标题就可以设置成14像素进行叠加，后面写的样式可以覆盖先前写的同名样式，相当灵活。<br />css1 1996.12.17<br />css2 1998.5.12 推行内容和表现分离，table布局开始落寞<br />css3 2001.5.23 主要包括盒子模型、列表模块、超链接方式、语言模块、背景和边框、文字特效、多栏布局等模块<br />css世界的诞生就是为图文信息展示服务的。

<a name="37a29af0"></a>
### 文档流
`流` 是CSS世界中的一种基本的定位和布局机制，可以理解为现实时间的一套物理规则， `流` 和现实世界的 `水流` 有异曲同工的表现。<br />eg:<br />拿 `div` 和 `span` 举例，分别是 `块级元素` 和 `内联元素`，分别对应容器里的水和木头，水是自动铺满容器的，木头依次排列。<br />所以， `流` 就是 CSS 世界中 引导元素排列和定位 的一条看不见的 `水流`。

<a name="4b3dca1d"></a>
#### 流是如何影响整个CSS世界的（CSS是如何设计的？）
整个CSS世界几乎就是围绕流来建立的。<br />那么流是如何影响整个CSS世界的？

1. 让 `HTML` 默认的表现符合 `流`，（CSS是继承层叠的）
1. 实现复杂的布局，就要打破坏 `流`
1. 改变流向，默认的流向是从左往右，自上而下

<a name="f4c99e12"></a>
#### 流体布局
指的是利用元素 `流` 的特性实现的各类布局效果。 `流` 本身都具有自适应性， 所以流体布局是也是有自适应性的。但是 `流体布局` ≠ `自适应布局`。自适应布局是对具有自适应特性的一类布局的统称。<br />所以，自适应布局 > 流体布局。

<a name="521d4cf5"></a>
### 未定义行为
在我们日常排版中，会遇到某些css在不同浏览器表现形式不一致的情况，会认为是某个浏览器的bug。<br />需要转变一下想法，web标准不会面面俱到的，规范描述未触达的地方，需要各个浏览器厂商自己去想象。<br />比如 [FireFox mousedown干掉:active实例页面](https://demo.cssworld.cn/2/2-1.php)<br />FireFox认为:active发生在mousedown事件之后

<a name="942e72cd"></a>
### 流、元素与基本尺寸
HTML通常分为两类：块级元素和内联元素

<a name="9d062fd9"></a>
#### 块级元素 （block-level element）
基本特征： 一个水平流上只能单独显示一个元素（也不用占满，比如 `<table>` 的宽度是由内容决定大小的，就算只有一列，也自己独占一行），多个块级元素则换行显示。<br />常见的块级元素有 `<div>` 、 `<li>` 、 `<table>` ，注意 `块级元素` 和 `display: block` 不是一个概念。<br />比如 `<li>` 元素的display的默认值是list-item， `<table>` 元素的display的默认值是table，但是他们都叫 `块级元素` ，因为他们都符合块级元素的基本特征。<br />作用：<br />由于 `块级元素` 具有换行特性，所以理论上它都可以配合  `clear` 属性来清除浮动带来的影响<br />clear属性是float的克星，官方属性解释： `元素盒子的边不能和前面的浮动元素相邻`。其值为 `none | left | right | both`<br />clear属性只有块级元素才有效的，而我们平常都会写

```javascript
.clear:after{
    content: '';
    display: table; // 也可以是block，或者list-item
    clear: both;
}
```

但是在实际开发中，我们不会用list-item来清除浮动

1. 字符比较多
2. 会出现不需要的项目符号，需要加list-style: none来去除
3. IE不支持伪元素的display的值为list-item

<a name="0ba47d0a"></a>
### 盒子的类型
盒子的类型种类是不断迭代的。

1. 块级盒子<br />
负责结构
1. 内联盒子<br />
负责内容，不管结构，比如 `<span>` 直接设置 height或者width是没有用的
1. 附加盒子（标记盒子marker box）<br />
比如，因为引入 `<ul>、<ol>、<li>`标签，需要显示项目符号，所以新增了一个新的 `display` 属性值 `list-item`，<br />
`<li>` 标签的默认的 `display` 属性值就是 `list-item`。<br />
所以在之前两个盒子的基础之上，所有 `块级元素` 都有一个 `主块级盒子`，而list-item除此之外还有一个 `附加盒子`。
1. 外在盒子和内在盒子<br />
display的值又出现了 `inline-block`，它的表现是穿着inline的皮藏着block的心，之前定义的盒子没法儿解释这个现象，<br />
然后导致每个元素都用两个盒子表示：`外在盒子和内在盒子` 。<br />
外在盒子负责元素是可以一行显示，还是只能换行显示（即体现元素是内联元素还是块级元素）；内在盒子负责宽高、内容呈现等。（内在盒子又称为“容器盒子”）<br />
对标兼容解释：<br />
`display: block` 的元素的盒子由外在的“块级盒子”和内在的“块级容器盒子”组成，<br />
`display: inline-block` 的元素则由外在的“内联盒子”和内在的“块级容器盒子”组成，<br />
`display: inline` 的元素则内外均是“内联盒子”。<br />
所以display属性值是inline-block的元素既能和图文一行显示，又能直接设置width/height （作用在内在盒子上），<br />
因为有两个盒子，外面的盒子是inline级别，里面的盒子是block级别。<br />
测试：<br />
`display: inline-table` 的表现形式<br />
外面是 `内联盒子` ，里面是 `table盒子`，得到一个可以和文字在一行显示的表格。

```javascript
HTML
和文字平起平坐的表格：
<table class="inline-table">
  <tr>
    <th>1</th>
    <th>2</th>
  </tr>
</table>
<div class="inline-table">
    <p>第1列</p>
    <p>第2列</p>
</div>
CSS
.inline-table {
    display: inline-table;
    width: 128px;
    margin-left: 10px;
    border: 1px solid #cad5eb;
}
.inline-table > p {
    display: table-cell;
}
```

<a name="fb01b166"></a>
### width/height作用在哪个盒子上
内在盒子上

<a name="d3b60f07"></a>
### width:auto;
css中，width的默认值是auto。但是它的表现形式有4种。

<a name="2fab8449"></a>
#### 充分利用可用空间
width: -webkit-fill-available;<br />比如 `<div>`、`<p>`这种块级元素的宽度默认是 100%于父级容器的

<a name="7722eda8"></a>
#### 收缩与包裹
width: -webkit-fit-content;<br />收缩到合适，比如浮动，绝对定位，inline-block元素或者table元素

<a name="a137409f"></a>
#### 收缩到最小
width: -webkit-min-content;<br />这个最容易出现在table-layout为automatic的[表格](https://demo.cssworld.cn/3/2-1.php)中<br />当每一列空间都不够的时候，文字能断就断，但是中文是可以随便断的，英文单词不能断

<a name="69d5825b"></a>
#### 超出容器限制
width: -webkit-max-content;<br />比如[内容很长的连续的英文和数字](https://demo.cssworld.cn/3/2-2.php)，或者内联元素被设置了 `white-space: nowrap;`

<a name="399d24f3"></a>
### CSS的尺寸
CSS尺寸分为内部尺寸和外部尺寸

<a name="955b5f26"></a>
#### 外部尺寸
表示宽度由外部元素决定（上面四种情况表现，只有第一种是外部尺寸）

<a name="bc06a996"></a>
##### 正常流宽度
在页面中随便扔一个 `<div>` 元素，其尺寸表现就会铺满容器，这个是块级元素的流特性。<br />所以在页面中丢一个 `<a>` 元素，设置其css `display` 属性的值为 `block`，就不用同时指定width，因为它会自动铺满容器。

```javascript
a {
    width: 100%;
    display: block;
}
```

上面的这个实例代码很常见，在写一个PC的导航栏， `.nav > a`，很多人会像上面代码示例一样这么写， 但实际上，这个width是没有出现的必要的。<br />并且，一旦给这个 `<a>` 设置了 `width` ，元素的流动反而会丢失。<br />所谓的流动性，并不是单纯表现形式上宽度100%这么简单，而是一种 `margin/border/padding` 和 `content` 内容区域自动分配水平空间的机制。<br />换言之：无论设置padding还是margin，其占据的空间一直不变，变化的就是content box的尺寸，这是典型的流体表现特性。

```javascript
<div class="nav">
  <a href="javascript:;" class="nav-a">导航栏一</a>
  <a href="javascript:;" class="nav-a">导航栏二</a>
</div>
<br>
<div class="nav">
  <a href="javascript:;" class="nav-a width-100">导航栏一</a>
  <a href="javascript:;" class="nav-a width-100">导航栏二</a>
</div>
.width-100 {
  width: 100%;
}
.nav {
  width: 240px;
  background-color: #cd0000;
}
.nav-a {
  display: block;
  margin: 0 10px;
  padding: 9px 10px;
  border-bottom: 1px solid #b70000;
  border-top: 1px solid #de3636;
  color: #fff;
}
.nav-a:first-child { border-top: 0; }
.nav-a + .nav-a + .nav-a { border-bottom: 0;}
```

事实上，第一个导航栏的审查元素宽度是220px，但是看上去是100%填充；第二个的导航栏审查元素宽度是260px

<a name="837f2f39"></a>
##### 格式化宽度
格式化宽度仅仅出现在 `绝对定位模型` 中，也就是出现在 `position: absolute/fixed`的元素中。<br />当然在默认情况下，绝对定位元素的宽度的表现应该是包裹性，即宽度由内部尺寸决定。<br />但是对于 `非替代元素`，当 `left/right` 或 `top/bottom` 对立方的属性值同时存在时，元素的宽度表现为 `格式化宽度`， 其宽度大小相对于最近的具有定位特性（position的属性值不是static）的祖先元素计算。<br />格式化宽度也具有完全的流体特性，也就是 `margin/border/padding` 和 `content` 内容区域自动分配水平（和垂直）空间。

```javascript
<div class="nav">
  <div class="nav-child"></div>
</div>
.nav {
  width: 200px;
  height: 200px;
  background-color: #cd0000;
  position: relative;
}
.nav-child {
  background:green;
  position: absolute;
  height: 100px;
  left: 0;
  right: 0;
}
```

<a name="ac28782a"></a>
#### 内部尺寸
表示宽度由内部元素决定

<a name="b98cfa19"></a>
##### 包裹性（包裹+自适应）
所谓 `自适应性`，指的是元素尺寸由内部元素元素决定，但永远小于 `包含块` 容器的尺寸（除非！容器尺寸比元素的首选最小宽度还小）<br />换言之，包裹性元素有个max-width: 100%;来罩着。

```javascript
例子一：
<div class="nav">
  匀速匀速匀速匀速匀速匀速匀速匀速匀速匀速匀速匀速匀速匀速匀速匀速匀速匀速匀速匀速匀速匀速匀速速匀速匀速匀速匀速匀速匀速匀速匀速匀速匀速匀速
</div>
<div class="nav">
121122212112221211222121122212112221211222121122212112221211222
</div>
.nav {
  background-color: #cd0000;
  display: inline-block;
}
// 如果nav容器的width写死，100px，再看看效果，就会触发上面的除非特例
```

类似的 `<button>` 也是有上述效果的，它是典型的 inline-block元素<br />容器的文字越多宽度越宽（内部尺寸特性），但是如果文字足够多，则会在容器的宽度处自动换行<br />作用：<br />需求： 页面某个模块的文字内容是动态的，设计师经常希望我们做处理，字数少的时候居中显示，文字超过一行的时候居左显示。

```javascript
<div class="box">
  <p class="content">文字内容-新增文字-新增文字-新增文字-新增文字-新增文字-新增文字</p>
</div>
<br>
<div class="box">
  <div class="content">文字</div>
</div>
.box {
  padding: 10px;
  width: 200px;
  background-color: #cd0000;
  text-align: center;
}
.content {
  display: inline-block;
  text-align: left;
}
```

<a name="97f40493"></a>
##### 首选最小宽度
指的是元素最适合的最小宽度。<br />假如上面例子的box元素宽度是由200px变成0，那么此时子元素content的宽度会是怎么样呢？

```javascript
<div class="box">
  <p class="content">文字内容-新增文字-新增文字-新增文字-新增文字-新增文字-新增文字</p>
</div>
<br>
<div class="box">
  <div class="content">文字</div>
</div>
.box {
  padding: 10px;
  width: 0;
  background-color: #cd0000;
  text-align: center;
}
.content {
  display: inline-block;
  text-align: left;
}
```

表现形式是，中文字会被一个个断开，英文字会按照单词来断开，如果设置了word-break: break-all;那么表现的也会像中文字一样。<br />此时content元素的宽度为尽可能的最小宽度。<br />eg：可以用来凹造型，排版一个凹凸造型

<a name="99b57d8c"></a>
##### 最大宽度
指的是元素可以有的最大宽度<br />"最大宽度"实际等于"包裹性"元素设置了 `white-space: nowrap;` 之后的宽度。<br />具体计算：如果内部没有块级元素，或者块级元素没有设定宽度值，则"最大宽度"实际上是最大的连续内联盒子的宽度。<br />最常见的作用[不用js的横向scroll](https://demo.cssworld.cn/3/2-7.php)

<a name="fd349c3c"></a>
### width值的作用
之前说过内在盒子负责宽高，内容呈现，所以可以说width是作用在 `内在盒子` 上的。<br />内在盒子的组成（盒模型）：margin（透明）+ border + padding + content<br />内在盒子又被分成了4个盒子，分别是 `content box` 、 `padding box` 、 `border-box` 、 `margin box`。<br />这四个盒子在CSS中也有对应的名字， `content-box` 、 `padding-box` 、 `border-box` ，就是没有 `margin-box` ，<br />因为 `margin的背景永远是透明的` ，margin设定具体的值，也不会影响本身的尺寸。因为它是透明的，看不见，没有感觉。因此作为 `box-sizing` 的属性值就没有了意义，然后就被抛弃了。<br />padding-box 之前火狐支持过<br />CSS2.1规定，content box 环绕着width 和 height给定的矩形。<br />换言之，就是说content box的宽高就是这个元素设置CSS的width、height的值。<br />`box-sizing` 的默认值是 `content-box` 也就好记了。<br />给元素设置了具体的width之后，会使流动性丢失，需要我们自己去计算宽度排版。

<a name="587c60d4"></a>
### height: 100%;
对于width属性，就算父元素的width为auto, 其百分比也是支持的。但是，对于height属性，如果父元素的height为auto，只要子元素还在文档流中，其百分比值是没有用的。

```javascript
// 在页面里插入一个<div>，然后满屏显示背景图
div {
    width: 100%; // 这是多余的，块级元素会自动铺满，流的特性
    height: 100%; // 这是无效的
    background: url(bg.jpg);
}
```

为什么 `height: 100%;` 是无效的？<br />因为CSS规范中，说如果包含块的高度没有显示指定（即高度由内容决定），并且该元素不是绝对定位，则计算值为auto。<br />auto * 百分比  = NAN<br />而宽度，规则中没有明确说是auto，需要各个浏览器自己去理解发挥（未定义行为）。不过所有的浏览器的理解表现恰巧是一样的。<br />因为宽度设置百分比，会拿真实的计算值 * 百分比。

<a name="ba6fecf9"></a>
#### 如何使height: 100%有效

1. 给父元素设置显示的高度值，height: 600px;
1. [使用绝对定位](https://demo.cssworld.cn/3/2-11.php)

```javascript
div {
    height: 100%;
    position: absolute;
}
// 绝对定位的宽高百分比是相对于 padding box的
// 非绝对定位则是相对于 content box的
```

<a name="a7d1b6da"></a>
### min-width/max-width和min-height/max-height
min-width/min-height的默认值是auto<br />max-width/max-height的默认值是none

<a name="5eb8ef43"></a>
#### 相互覆盖规则

1. 超越!important

```javascript
<div style="width: 300px !important;">
div {
    max-width: 200px;
}
```

2. 超越最大

```javascript
div {
    min-width: 1300px;
    max-width: 1000px;
}
```

需求：css做不确定高度的div展开收起动画

```javascript
.element {
  height: 0;
  overflow: hidden;
  transition: height .25s;
}
.element.active {
  height: auto; // 没有动画，因为auto无法计算
}
```

```javascript
.element {
  max-height: 0;
  overflow: hidden;
  transition: max-height .25s;
}
.element.active {
  max-height: 666px;  // 一个相对足够大的最大高度
}
```
