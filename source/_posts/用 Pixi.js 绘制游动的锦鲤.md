---
updated: '2016-01-18 00:00:00'
categories: code
excerpt: "Pixi.js\n\_是一个 Javascript 的 2D 图像引擎，它采用 WebGL 加速 可实现惊人的绘制效率，并且它对 2D 渲染、动画、特效等进行了封装，十分易于使用，下面我们来一步步用 Pixi 实现游动的鲤鱼的效果。"
date: '2016-01-18 00:00:00'
tags:
  - JavaScript
urlname: pixi-koi
title: 用 Pixi.js 绘制游动的锦鲤
---

[Pixi.js](http://www.pixijs.com/) 是一个 Javascript 的 2D 图像引擎，它采用 WebGL 加速 可实现惊人的绘制效率，并且它对 2D 渲染、动画、特效等进行了封装，十分易于使用，下面我们来一步步用 Pixi 实现游动的鲤鱼的效果。


在 Pixi 中，动画的绘制和 WebGL 一样是通过 requestAnimationFrame 实现的，在每一帧清空整个场景，然后再将对应的精灵绘制到对应位置上即可实现动画效果，首先我们要绘制一条静态的鱼，考虑到后面需要让它动起来，我们将它分为几部分：身体和左右鳍，如图：


![20200601180253.png](https://s.z4none.me/blog/26345ddda40d1dce6e59dfe8d851b8bb.png)


[demo 1](https://g.z4none.me/fish/1.html)


这里为了示意，将图片设为半透明


然后在渲染函数中根据时间（tick）将左右鳍动起来，这里通过改变 yScale 即可实现鱼鳍划动的效果


![20200601180443.gif](https://s.z4none.me/blog/3de6cb3b73f8b653791deff3c23a2f40.gif)


[demo 2](https://g.z4none.me/fish/2.html)


现在鱼显得有些呆板，现实中的鱼在水中游动时是会扭动身体的，扭动的效果可以通过 PIXI.mesh.Rope 实现， 相关例子：[http://pixijs.io/examples/#/basics/textured-mesh.js](http://pixijs.io/examples/#/basics/textured-mesh.js)


对应我们的鱼不需要取那么多控制点，这里只取了 4 个控制点，在扭动的同时，鱼鳍的位置也要跟随改变


![20200601180809.gif](https://s.z4none.me/blog/abe87b1e0a5bbf049db64cca6aa550c6.gif)


[demo 3](https://g.z4none.me/fish/3.html)


接下来就是让这个固定的鱼在场景中游动起来了，鱼游动的规律可以根据需要自己建模来模拟，我这里实现了一个 Demo 只是简单的让它们游向随机点。


[final live demo](https://g.z4none.me/fish/)


[https://github.com/z4none/fish](https://github.com/z4none/fish)

