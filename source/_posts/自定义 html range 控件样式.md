---
updated: '2022-08-29 08:00:00'
categories: code
excerpt: 当将 HTML input 元素的 type 定义成 range 时，其形式类似 Windows 的 Trackbar，或者俗称 Slider
date: '2022-08-29 08:00:00'
tags:
  - Web
urlname: html-range-input-style
title: 自定义 html range 控件样式
---

当将 HTML input 元素的 type 定义成 range 时，其形式类似 Windows 的 Trackbar，或者俗称 Slider，一般是这个样子：


![Untitled.png](https://s.z4none.me/blog/2acc1ca7be8cc183c74e263b27ed2c30.png)


在现代浏览器中，它的样式各种各样：


![Untitled.png](https://s.z4none.me/blog/8a3866ae273c8f1c93e5f47e7feffe83.png)


为了让它符合页面的需要，有时我们需要对它的样式进行自定义。


经过研究，有两种方法可以自定义 thumb 左右的 tracker 为不同颜色：


## 1. box-shadow + overflow: hidden


在 slider-thumb 上定义向左的投影，然后在 slider 上定义 overflow: hidden 将外部的多余部分隐藏，这样看起来就好像是左侧的 tracker 是自定义了颜色的。


![Untitled.png](https://s.z4none.me/blog/dbf8765342b62d6f10c22f689f1db58b.png)


这种方法的优点是采用纯 CSS，实现简单，缺点是 slider-thumb 会被裁切，不能超出 tracker 范围。


[https://codepen.io/z4none/pen/zYjxbwQ](https://codepen.io/z4none/pen/zYjxbwQ)


## 2. linear-gradient + Javascript


在 input[type=range] 上定义一个渐变背景色，然后通过 Javascript 监听 input 事件，根据控件值修改元素的 backgroundSize， 从而实现左右 tracker 为不同颜色，效果如下：


![Untitled.png](https://s.z4none.me/blog/afae326b94d6c876c46b985d0d64a762.png)


[https://codepen.io/z4none/pen/ExLZbxq](https://codepen.io/z4none/pen/ExLZbxq)

