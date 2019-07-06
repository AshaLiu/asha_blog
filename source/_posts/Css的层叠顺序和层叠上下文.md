---
title: Css的层叠顺序和层叠上下文
date: 2018-04-27 14:43:44
tags: css z-index
---
### 前言
前两天这不是看了下`复合层`的概念么，然后在看别人的博客的时候，看到了这样的一句话，说是`比如一个fix在页面顶部的固定不变的导航header，在页面内容某个区域repaint时，整个屏幕包括fix的header也会被重绘。`
我觉得挺奇怪的，fixed不是已经浮起来脱离文档流了么，又不跟人家一个层，怎么会受影响呢。后来作者解释了之后，查了相关资料才明白。

### 层叠顺序（stacking level）
啥叫 `层叠顺序`，就是在空间Z轴的层叠关系。比较常见的是我们的`z-index`的使用。
举个例子：
```html
<div class="container">
    <div class="inline-blockA">#divA display:inline-block</div>
    <div class="floatB"> #divB float:left</div>
</div>
```
```css
.container{
    position:relative;
    background:#ddd;
}
.container > div{
    width:200px;
    height:200px;
}
.floatB{
    float:left;
    background-color:deeppink;
}
.inline-blockA{
    display:inline-block;
    background-color:yellowgreen;
    margin-left:-100px;
}
```

你看这两个方块产生了overlap，那你猜A在上还是B在上了。自己去实验下。
正确答案是A在上面。是不是很吃惊，按照常理来理解，float都浮起来了。。。

#### 7阶层叠水平规则
英文版：
1. the background and borders of the element forming the stacking context.
2. the child stacking contexts with negative stack levels (most negative first).
3. the in-flow, non-inline-level, non-positioned descendants.
4. the non-positioned floats.
5. the in-flow, inline-level, non-positioned descendants, including inline tables and inline blocks.
6. the child stacking contexts with stack level 0 and the positioned descendants with stack level 0.
7. the child stacking contexts with positive stack levels (least positive first).

中文版：（7在上1在下）
1. 形成堆叠上下文环境的元素的背景与边框
2. 拥有负`z-index`的子堆叠上下文元素 （负的越高越堆叠层级越低）
3. 正常流式布局，非`inline-block`，无`position`定位`static除外`的子元素
4. 无 position 定位（static除外）的 float 浮动元素
5. 正常流式布局，`inline-block`元素，无`position`定位`static除外`的子元素`包括display:table 和 display:inline`
6. 拥有`z-index:0`的子堆叠上下文元素
7. 拥有正`z-index:`的子堆叠上下文元素（正的越低越堆叠层级越低）

这就可以解释上面的例子为什么A没有"浮起来"，却还是在B上面

### 层叠上下文（stacking context）
事情是不会这么顺利的，我们来修改下题目：
```css
.container{
    position:relative;
    background:#ddd;
}
.container > div{
    width:200px;
    height:200px;
    opacity:0.9; // 注意，这里增加一个opacity
}
.floatB{
    float:left;
    background-color:deeppink;
}
.inline-blockA{
    display:inline-block;
    background-color:yellowgreen;
    margin-left:-100px;
}
```
猜答案，这回是B在上面了。为什么呢？这就涉及另一个概念`层叠上下文`了。
来一个比喻，整个html像是一个整个天下，其中有各路诸侯，诸侯之间有等级之分，诸侯里面还有各个幕僚啥的，这些人呢也有等级之分，毕竟制度嘛，肯定有大小之分。（最近看三国比较多，捂脸）
这个`大小之分`就是`层叠顺序`，这个`诸侯`就是`层叠上下文`。

看上面的例子，A、B都搞了个`opacity`加成，使这个A、B生成了`堆叠上下文`。这个时候，层叠顺序就是要看哪个诸侯厉害了，我的主人厉害，我就是要骑在你头上！
看来这个加成很好用啊，那怎么样才能让我获得这个加成呢？

1. 根元素 (HTML),
2. z-index 值不为 "auto"的 绝对/相对定位，
3. 一个 z-index 值不为 "auto"的 flex 项目 (flex item)，即：父元素 display: flex|inline-flex，
4. opacity 属性值小于 1 的元素（参考 the specification for opacity），
5. transform 属性值不为 "none"的元素，
6. mix-blend-mode 属性值不为 "normal"的元素，
7. filter值不为“none”的元素，
8. perspective值不为“none”的元素，
9. isolation 属性被设置为 "isolate"的元素，
10. position: fixed
11. 在 will-change 中指定了任意 CSS 属性，即便你没有直接指定这些属性的值
12. -webkit-overflow-scrolling 属性被设置 "touch"的元素

要是两个人都加成了，那怎么比较谁厉害呢？继续走上面的那个层叠顺序的规则。

### 我来谈谈
突然想起来以前刚排版的时候，会碰到"拼爹的absolute的z-index"，排版排不出我想要的正确的层叠的顺序。后来看了大佬张鑫旭的才知道"拼爹"这个概念，就记住了这个拼爹的比喻。
现在想想，这不就是不同堆栈上下文看层叠么。恍然大悟。知道原理才更加领悟。

希望今年博客高产。

### 参考资料
[层叠顺序与堆栈上下文知多少](https://www.cnblogs.com/coco1s/p/5899089.html)