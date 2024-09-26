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


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/2cf9c9be-2d7d-4418-959b-a46a80530dde/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T043407Z&X-Amz-Expires=3600&X-Amz-Signature=4ed05dc93f94c5dc49c8079603fbfc7f5e751ae9c783a2be1d0f96156c4118ed&X-Amz-SignedHeaders=host&x-id=GetObject)


在现代浏览器中，它的样式各种各样：


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/61aed52c-96f2-4ab0-b0e9-7b38fa618f39/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T043407Z&X-Amz-Expires=3600&X-Amz-Signature=267dd071c0ffe589c78369a734b49419e95e05c2f38282f934c1eee86b714073&X-Amz-SignedHeaders=host&x-id=GetObject)


为了让它符合页面的需要，有时我们需要对它的样式进行自定义。


经过研究，有两种方法可以自定义 thumb 左右的 tracker 为不同颜色：


## 1. box-shadow + overflow: hidden


在 slider-thumb 上定义向左的投影，然后在 slider 上定义 overflow: hidden 将外部的多余部分隐藏，这样看起来就好像是左侧的 tracker 是自定义了颜色的。


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/a3f7e700-1140-4260-b7e6-4e824e951055/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T043407Z&X-Amz-Expires=3600&X-Amz-Signature=f350d97f307a487c4e210417b7c42b99a0826b15780276e6810c379287948910&X-Amz-SignedHeaders=host&x-id=GetObject)


这种方法的优点是采用纯 CSS，实现简单，缺点是 slider-thumb 会被裁切，不能超出 tracker 范围。


[https://codepen.io/z4none/pen/zYjxbwQ](https://codepen.io/z4none/pen/zYjxbwQ)


## 2. linear-gradient + Javascript


在 input[type=range] 上定义一个渐变背景色，然后通过 Javascript 监听 input 事件，根据控件值修改元素的 backgroundSize， 从而实现左右 tracker 为不同颜色，效果如下：


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/69ea714c-ba48-4c94-966a-071244837ab6/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T043407Z&X-Amz-Expires=3600&X-Amz-Signature=600d9b96fb380d26e7a014980bf0562de7d839c9afa75de6e37828e398dbeee1&X-Amz-SignedHeaders=host&x-id=GetObject)


[https://codepen.io/z4none/pen/ExLZbxq](https://codepen.io/z4none/pen/ExLZbxq)

