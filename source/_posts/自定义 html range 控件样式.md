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


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/2cf9c9be-2d7d-4418-959b-a46a80530dde/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T050930Z&X-Amz-Expires=3600&X-Amz-Signature=c36b9322ea5ce5e40a77c9614a1f2736249973611672cae41d3f774e2732bd56&X-Amz-SignedHeaders=host&x-id=GetObject)


在现代浏览器中，它的样式各种各样：


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/61aed52c-96f2-4ab0-b0e9-7b38fa618f39/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T050930Z&X-Amz-Expires=3600&X-Amz-Signature=0c2b84016fa8b567aca4da7a8bcbb8863b769503c395b9a0a7f27ca6dec40b00&X-Amz-SignedHeaders=host&x-id=GetObject)


为了让它符合页面的需要，有时我们需要对它的样式进行自定义。


经过研究，有两种方法可以自定义 thumb 左右的 tracker 为不同颜色：


## 1. box-shadow + overflow: hidden


在 slider-thumb 上定义向左的投影，然后在 slider 上定义 overflow: hidden 将外部的多余部分隐藏，这样看起来就好像是左侧的 tracker 是自定义了颜色的。


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/a3f7e700-1140-4260-b7e6-4e824e951055/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T050930Z&X-Amz-Expires=3600&X-Amz-Signature=c3de25ac83c60f90bc539f21269c7f360ae34f8a9d856e71ac1292bdd93077ff&X-Amz-SignedHeaders=host&x-id=GetObject)


这种方法的优点是采用纯 CSS，实现简单，缺点是 slider-thumb 会被裁切，不能超出 tracker 范围。


[https://codepen.io/z4none/pen/zYjxbwQ](https://codepen.io/z4none/pen/zYjxbwQ)


## 2. linear-gradient + Javascript


在 input[type=range] 上定义一个渐变背景色，然后通过 Javascript 监听 input 事件，根据控件值修改元素的 backgroundSize， 从而实现左右 tracker 为不同颜色，效果如下：


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/69ea714c-ba48-4c94-966a-071244837ab6/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T050930Z&X-Amz-Expires=3600&X-Amz-Signature=165ad84c58cfb2b564406827a4b5be074cb42bf33adf6c405fc055f74645a22d&X-Amz-SignedHeaders=host&x-id=GetObject)


[https://codepen.io/z4none/pen/ExLZbxq](https://codepen.io/z4none/pen/ExLZbxq)

