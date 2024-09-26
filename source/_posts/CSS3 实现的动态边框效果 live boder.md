---
updated: '2015-12-13 08:00:00'
categories: code
excerpt: |-
  最近 Google+ 进行了改版，整体效果如何我就不吐槽了.
  有些设计上的细节比较有意思，比如这个后台载入的提示:
  通过开发者工具分析可以看出来，它是通过 CSS3 animation 让一个 div 循环移动实现的
  于是我也来尝试实现了一下这个效果，并封装成 jQuery 插件
date: '2015-12-13 08:00:00'
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


	![20190830162731.gif](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/bf313cd9-b164-40d0-b024-fda8da5d0d8a/20190830162731.gif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T042916Z&X-Amz-Expires=3600&X-Amz-Signature=3030b92e8f6118e2672c2f525c5dcbc40459a857ad765914b76940e4a1c151c8&X-Amz-SignedHeaders=host&x-id=GetObject)


	[https://github.com/z4none/jquery-live-border/](https://github.com/z4none/jquery-live-border/)

