---
title: 推进webp格式的图片 ---APP篇
date: 2018-07-30 13:36:38
tags: webp
---
### 前言
作为一个前端领域的人，人家问你一个对于图片这块儿有什么地方可以注意提升的吗？
回答：
1. 可以根据业务不同，可以再服务器新增存一张图片的几个替身，比如`-100.jpg`，`-300.jpg`等；
2. 可以使用`iconfont`；
3. 可以`图片精灵`
4. 可以`懒加载`
5. 可以`webp`
6. 可以上`cdn`
7. 等等等等

最近手头有点空，觉得不能荒废时间，要提升自己不断做尝试。于是我就准备搞事情，推进公司`webp`的使用，为啥叫推进呢，因为这个事情要服务端，app，h5，ui多个部门一起协作。
事情吧，要是只是自己做，可以闷不吭声，默默地啥时候做好啥时候上，但是这种涉及到跨部门协作的事情，就要大家一起合体发力了。

### 概念
概念的话网上一搜有很多，是一种由谷歌推出的同时提供了有损压缩与无损压缩的图片文件格式。在质量相同的情况下，`WebP`格式图像的体积要比`JPEG`格式图像小40%。
这么一听，感觉还挺好的。比较现在国内主流是移动端，移动端最耗流量的就是图片啊，视频啊什么的。

了解并且推进使用需要知道这样东西是优缺点，是否适合我们。

缺点：缺点很明显，新技术嘛，兼容性哇，这个在`can i use上`可以查一下。不过这个问题不大，可以`渐进增强`。

### 项目分析
前面讲完了一堆，确定我们想做这个事情，下面来分析我的项目。
目前公司项目前端涉及到的端是，`app混合式开发`，`m站`，`pc站`，`小程序`。由于我公司的主流产品是混合式开发，所以呢我先从`app混合式开发`项目开始。
分析分析：
1. 这个`车商端项目`，采用的是混合式开发，ios今年已经升级为`wkWebView`，安卓采用的是`webkit内核`。
2. 图片来源，一个是UI做的，一个是动态的用户上传的。
3. 图片用途，一个是背景图，一个是img标签图
4. 上面讲的是前端使用范畴，还有安卓ios自己使用webp的
一个个来攻克

### 还没写好，占坑



