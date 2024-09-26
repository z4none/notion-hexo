---
updated: '2016-01-18 08:00:00'
categories: code
excerpt: "Pixi.js\n\_是一个 Javascript 的 2D 图像引擎，它采用 WebGL 加速 可实现惊人的绘制效率，并且它对 2D 渲染、动画、特效等进行了封装，十分易于使用，下面我们来一步步用 Pixi 实现游动的鲤鱼的效果。"
date: '2016-01-18 08:00:00'
tags:
  - JavaScript
urlname: pixi-koi
title: 用 Pixi.js 绘制游动的锦鲤
---

[Pixi.js](http://www.pixijs.com/) 是一个 Javascript 的 2D 图像引擎，它采用 WebGL 加速 可实现惊人的绘制效率，并且它对 2D 渲染、动画、特效等进行了封装，十分易于使用，下面我们来一步步用 Pixi 实现游动的鲤鱼的效果。


在 Pixi 中，动画的绘制和 WebGL 一样是通过 requestAnimationFrame 实现的，在每一帧清空整个场景，然后再将对应的精灵绘制到对应位置上即可实现动画效果，首先我们要绘制一条静态的鱼，考虑到后面需要让它动起来，我们将它分为几部分：身体和左右鳍，如图：


![20200601180253.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/02eb2bec-b23c-4723-9328-50d6f1d1f87c/20200601180253.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T043411Z&X-Amz-Expires=3600&X-Amz-Signature=3b601ca3fff08ab68adbdb73f08701d1f7bfd8dea3a1bedbe8c19ec2d9d02a06&X-Amz-SignedHeaders=host&x-id=GetObject)


[demo 1](https://g.z4none.me/fish/1.html)


这里为了示意，将图片设为半透明


然后在渲染函数中根据时间（tick）将左右鳍动起来，这里通过改变 yScale 即可实现鱼鳍划动的效果


![20200601180443.gif](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/f5213ce4-9d03-4628-b0e1-1bc94f04523d/20200601180443.gif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T043411Z&X-Amz-Expires=3600&X-Amz-Signature=c0b3a48023eac8e7f83038f53cc162a9c4b9da11cccf086f4aaedcc387af1112&X-Amz-SignedHeaders=host&x-id=GetObject)


[demo 2](https://g.z4none.me/fish/2.html)


现在鱼显得有些呆板，现实中的鱼在水中游动时是会扭动身体的，扭动的效果可以通过 PIXI.mesh.Rope 实现， 相关例子：[http://pixijs.io/examples/#/basics/textured-mesh.js](http://pixijs.io/examples/#/basics/textured-mesh.js)


对应我们的鱼不需要取那么多控制点，这里只取了 4 个控制点，在扭动的同时，鱼鳍的位置也要跟随改变


![20200601180809.gif](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/e2723aca-db87-4721-8cd6-43770588309c/20200601180809.gif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T043411Z&X-Amz-Expires=3600&X-Amz-Signature=618cbeefcfbae4e6dc4cb4bf7b0bf487d743e7d96be05831d5e8c8a84a1db33a&X-Amz-SignedHeaders=host&x-id=GetObject)


[demo 3](https://g.z4none.me/fish/3.html)


接下来就是让这个固定的鱼在场景中游动起来了，鱼游动的规律可以根据需要自己建模来模拟，我这里实现了一个 Demo 只是简单的让它们游向随机点。


[final live demo](https://g.z4none.me/fish/)


[https://github.com/z4none/fish](https://github.com/z4none/fish)

