---
updated: '2015-12-13 00:00:00'
categories: code
excerpt: |-
  最近 Google+ 进行了改版，整体效果如何我就不吐槽了.
  有些设计上的细节比较有意思，比如这个后台载入的提示:
  通过开发者工具分析可以看出来，它是通过 CSS3 animation 让一个 div 循环移动实现的
  于是我也来尝试实现了一下这个效果，并封装成 jQuery 插件
date: '2015-12-13 00:00:00'
tags:
  - JavaScript
urlname: css3-live-border
title: CSS3 实现的动态边框效果 live boder
---

最近 Google+ 进行了改版，整体效果如何我就不吐槽了.


有些设计上的细节比较有意思，比如这个后台载入的提示:


通过开发者工具分析可以看出来，它是通过 CSS3 animation 让一个 div 循环移动实现的


于是我也来尝试实现了一下这个效果，并封装成 jQuery 插件，


通过举一反三，还可以实现动态环绕的边框效果：


目前的封装很简单，使用起来也很简单(依赖 jQuery)

- 引入相关文件

	```text
	<link rel="stylesheet" href="live-border.css"/><script type="text/javascript" src="live-border.js"></script>
	```

- 初始化

	```text
	$("#id-btn1").liveBorder();
	$("#id-btn2").liveBorder({
	top: true,
	right: true,
	bottom: true,
	left: true
	});
	```


	![20190830162731.gif](https://s.z4none.me/blog/55108bac8de550c13141b5ada2d27f14.gif)


	[https://github.com/z4none/jquery-live-border/](https://github.com/z4none/jquery-live-border/)

