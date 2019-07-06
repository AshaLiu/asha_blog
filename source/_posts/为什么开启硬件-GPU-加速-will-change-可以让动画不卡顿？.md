---
title: 为什么开启硬件(GPU)加速'will-change'可以让动画不卡顿？
date: 2018-04-24 09:09:21
tags: css 性能 composite
---
### 卡顿
目前大多数设备的屏幕刷新率为`60 次/秒`。因此，如果在页面中有一个动画或渐变效果，或者用户正在滚动页面，那么浏览器渲染动画或页面的每一帧的速率也需要跟设备屏幕的刷新率保持一致。
其中每个帧的预算时间仅比` 16 毫秒多一点 (1 秒/ 60 = 16.66 毫秒)`。但实际上，浏览器有整理工作要做，因此您的所有工作需要在 10 毫秒内完成。如果无法符合此预算，帧率将下降，并且内容会在屏幕上抖动。 此现象通常称为卡顿，会对用户体验产生负面影响。

### 浏览器渲染过程
这个其实在之前的文章也有提到过，浏览器在下载完资源之后的过程详见[虚拟DOM](https://ashaliu.github.io/2017/11/26/%E8%99%9A%E6%8B%9FDOM/)

今天来讲讲其他的
![流程图](/images/article/performance/1.jpg)
* JavaScript: 一般来说，我们会使用 JavaScript 来实现一些视觉变化的效果。比如做一个动画或者往页面里添加一些 DOM 元素等。
* Style: 计算样式，这个过程是根据 CSS选择器，对每个 DOM 元素匹配对应的 CSS 样式。这一步结束之后，就确定了每个 DOM 元素上该应用什么 CSS 样式规则。
* Layout: 布局，上一步确定了每个 DOM 元素的样式规则，这一步就是具体计算每个 DOM 元素最终在屏幕上显示的大小和位置。web 页面中元素的布局是相对的，因此一个元素的布局发生变化，会联动地引发其他元素的布局发生变化。比如，<body> 元素的宽度的变化会影响其子元素的宽度，其子元素宽度的变化也会继续对其孙子元素产生影响。因此对于浏览器来说，布局过程是经常发生的。
* Paint: 绘制，本质上就是填充像素的过程。包括绘制文字、颜色、图像、边框和阴影等，也就是一个 DOM 元素所有的可视效果。一般来说，这个绘制过程是在多个层上完成的。
* Composite: 渲染层合并，由上一步可知，对页面中 DOM 元素的绘制是在多个层上进行的。在每个层上完成绘制过程之后，浏览器会将所有层按照合理的顺序合并成一个图层，然后显示在屏幕上。对于有位置重叠的元素的页面，这个过程尤其重要，因为一旦图层的合并顺序出错，将会导致元素显示异常。

实际的场景下，会有三种常见的渲染流程：
1. JavaScript -> Style -> Layout -> Paint -> Composite (JS/CSS -> 计算样式 -> 布局 -> 绘制 -> 渲染出合并)
2. JavaScript -> Style ->Paint -> Composite (JS/CSS -> 计算样式 -> 绘制 -> 渲染出合并)
3. JavaScript -> Style -> Composite (JS/CSS -> 计算样式 -> 渲染出合并)

什么操作会触发重排和重绘看[这里](https://ashaliu.github.io/2016/12/14/DOM%E7%9A%84%E9%87%8D%E6%8E%92%E5%92%8C%E9%87%8D%E7%BB%98/)
今天的重点是在这个`Composite`

### 渲染层

![流程图](/images/article/performance/2.png)
在浏览器中，页面内容是存储为由 Node 对象组成的树状结构，也就是 DOM 树。每一个 HTML element 元素都有一个 Node 对象与之对应，DOM 树的根节点永远都是 Document Node。这一点相信大家都很熟悉了，但其实，从 DOM 树到最后的渲染，需要进行一些转换映射。

#### 从Nodes到LayoutObjects
DOM 树中得每个 Node 节点都有一个对应的 LayoutObject 。LayoutObject 知道如何在屏幕上 paint Node 的内容。

#### 从LayoutObjects到PaintLayers
一般来说，拥有相同的坐标空间的 LayoutObjects，属于同一个`渲染层`（PaintLayer）。

##### PaintLayers分类
PaintLayer 最初是用来实现 stacking contest（层叠上下文），以此来保证页面元素以正确的顺序合成（composite），这样才能正确的展示元素的重叠以及半透明元素等等。因此满足形成层叠上下文条件的 LayoutObject 一定会为其创建新的渲染层，
当然还有其他的一些特殊情况，为一些特殊的 LayoutObjects 创建一个新的渲染层，比如 overflow != visible 的元素。
根据创建 PaintLayer 的原因不同，可以将其分为常见的 3 类：

1. NormalPaintLayer
	* 根元素(HTML)
	* 有明确的定位属性（relative、fixed、sticky、absolute）
	* 透明的(opacity小于1)
	* 有 CSS滤镜(filter)
	* 有 CSS mask 属性
	* 有 CSS mix-blend-mode 属性（不为 normal)
	* 有 CSS transform 属性（不为 none）
	* backface-visibility 属性为 hidden
	* 有 CSS reflection 属性
	* 有 CSS column-count 属性（不为 auto）或者 有 CSS column-width 属性（不为 auto）
	* 当前有对于 opacity、transform、fliter、backdrop-filter 应用动画

2. OverflowClipPaintLayer
	* overflow不为visible(默认值)

3. NoPaintLayer
	* 不需要paint的PaintLayer，比如一个没有视觉属性(背景、颜色、阴影等)的空div

满足以上条件的LayoutObject会拥有独立的渲染层，而其他的LayoutObject则和其第一个拥有渲染层的父元素共用一个。

### GraphicsLayer
上面介绍了PaintLayer，有些特殊的PaintLayer会被认为是`合成层(Compositing Layers)`，合成层拥有单独的GraphicsLayer（图形层），
而其他不是合成层的渲染层，则和其第一个拥有`GraphicsLayer`父层公用一个。

每个`GraphicsLayer`都有一个`GraphicsContext`，`GraphicsContext`会负责输出该层的位图，位图是存储在共享内存中，作为`纹理`上传到 GPU 中，最后由 GPU 将多个位图进行合成，然后 draw 到屏幕上，此时，我们的页面也就展现到了屏幕上。

注：渲染层提升为合成层有一个先决条件，该渲染层必须是`SelfPaintingLayer`（基本可认为是上文介绍的`NormalPaintLayer`）。以下所讨论的渲染层提升为合成层的情况都是在该渲染层为`SelfPaintingLayer`前提下的。

太难了，太复杂了，我感觉我讲不下去了。

#### 如何变成合成层
1. 3D 或透视变换(perspective transform) CSS 属性
2. 使用加速视频解码的`<video>`元素 拥有 3D
3. (WebGL) 上下文或加速的 2D 上下文的`<canvas>`元素
4. 混合插件(如 Flash)
5. 对自己的 opacity 做 CSS动画或使用一个动画变换的元素
6. 拥有加速 CSS 过滤器的元素
7. 元素有一个包含复合层的后代节点(换句话说，就是一个元素拥有一个子元素，该子元素在自己的层里)
8. 元素有一个z-index比自己低且包含一个复合层的兄弟元素(换句话说就是该元素在复合层上面渲染)，或者是假设重叠

这里[无线性能优化：Composite](http://taobaofed.org/blog/2016/04/25/performance-composite/)说的非常详细，也都有demo

#### 隐式合成
This is called implicit compositing: One or more non-composited elements that should appear above a composited one in the stacking order are promoted to composite layers — i.e. painted to separate images that are then sent to the GPU.
大概意思就是：一个或多个非合成元素应出现在[堆叠顺序](https://www.cnblogs.com/coco1s/p/5899089.html)上的合成元素之上(z-index)，会被提升到合成层，即被绘制成分离的图像，然后将图像交给 GPU 处理。
就是上面的第八条

#### 合成层的优点
一旦paintLayer提升为了合成层就会有自己的绘图上下文，并且会开启硬件加速，有利于性能提升。
1. 合成层的位图，会交由 GPU 合成，比 CPU 处理要快
2. 当需要 repaint 时，只需要 repaint 本身，不会影响到其他的层
3. 对于 transform 和 opacity 效果，不会触发 layout 和 paint

#### 注意
1. 提升到合成层后合成层的位图会交GPU处理，但请注意，仅仅只是合成的处理（把绘图上下文的位图输出进行组合）需要用到GPU，生成合成层的位图处理（绘图上下文的工作）是需要CPU
2. 当需要repaint的时候可以只repaint本身，不影响其他层，但是paint之前还有style， layout,那就意味着即使合成层只是repaint了自己，但style和layout本身就很占用时间
3. 仅仅是transform和opacity不会引发layout 和paint，其他的属性不确定。

总结合成层的优势：一般一个元素开启硬件加速后会变成合成层，可以独立于普通文档流中，改动后可以避免整个页面重绘，提升性能。

### 性能优化点
1. 提升动画效果的元素 合成层的好处是不会影响到其他元素的绘制，因此，为了减少动画元素对其他元素的影响，从而减少paint，我们需要把动画效果中的元素提升为合成层。
提升合成层的最好方式是使用 CSS 的`will-change`属性。从上一节合成层产生原因中，可以知道`will-change 设置为opacity、transform、top、left、bottom、right 可以将元素提升为合成层。`
2. 使用`transform`或者`opacity`来实现动画效果, 这样只需要做合成层的合并就好了。
3. 减少绘制区域 对于不需要重新绘制的区域应尽量避免绘制，以减少绘制区域，而对于固定不变的区域，我们期望其并不会被重绘，因此可以通过之前的方法，将其提升为独立的合成层。减少绘制区域，需要仔细分析页面，区分绘制区域，减少重绘区域甚至避免重绘。

### 合成层过犹不及
1. 合成层占用内存的问题
2. 层爆炸，由于某些原因可能导致产生大量不在预期内的合成层，虽然有浏览器的层压缩机制，但是也有很多无法进行压缩的情况，
这就可能出现层爆炸的现象（简单理解就是，很多不需要提升为合成层的元素因为某些不当操作成为了合成层）。解决层爆炸的问题，最佳方案是打破 overlap 的条件，
也就是说让其他元素不要和合成层元素重叠。简单直接的方式：`使用3D硬件加速提升动画性能时，最好给元素增加一个z-index属性，人为干扰合成的排序，可以有效减少chrome创建不必要的合成层，提升渲染性能，移动端优化效果尤为明显。`

### 只触发composite
这里就该扯回去了，最好的就是希望不涉及布局和绘制，只需要合成层合并。目前有两个特殊属性是可以支持的，`transforms`和`opacity`。所以说动画如果可以的话，用这两个属性。
这里有个[网站](https://csstriggers.com/)可以做参考。

注意： 元素要提升为合成层后，`transform`和`opacity`才不会触发repaint。

### 总结
说道最后，其实文章的题目有点以偏概全了，`will-change`只是其中一种情况而已。
另外google浏览器还有很多`devTools`可以图形化的观察，比如`performance`,`layers`,`rendering`

### 参考
这些挺深入的我感觉，平时也没什么时间机会接触这些，花了我挺多时间吃下去。其实这些东西还可以衍生出去很多，比如如何一步步优化，平时要注意些什么，等之后有时间再看看吧。
[无线性能优化：Composite](http://taobaofed.org/blog/2016/04/25/performance-composite/)
[浏览器渲染流程&Composite（渲染层合并）简单总结](https://segmentfault.com/a/1190000014520786)
[详谈层合成（composite）](https://juejin.im/entry/59dc9aedf265da43200232f9)
[CSS trigger](https://csstriggers.com/)
