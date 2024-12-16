---
updated: '2024-12-16 00:00:00'
categories: code
excerpt: 在测试环境中，用户常常面临因长时间未操作而自动退出登录的问题。为了解决这一困扰，我开发了一个简单的 Bookmarklet，它是一段保存在浏览器书签中的 JavaScript 代码。用户只需将代码添加到书签栏，点击即可自动定时刷新页面，保持登录状态，极大地方便了长时间在线的测试工作。
date: '2024-12-16 00:00:00'
tags:
  - JavaScript
urlname: auto-refresh-bookmarklet
title: 自动定时刷新页面 bookmarklet
---

# 背景


这是在我开发中遇到的一个问题，在测试环境中，我需要从外部系统的一个接口定时抓取数据，这个系统需要登录才能访问，登陆后如果一定时间没有操作则会自动退出登陆状态。为此，我需要一个简单的方式，在测试环境中自动定时刷新页面，从而避免因超时而导致的登录失效。


# 介绍


于是，我写了一个简单的 Bookmarklet 来完成这个工作，Bookmarklet 是一种保存在浏览器书签中的 JavaScript 代码片段，可以为网页添加额外功能。用户只需将该工具的代码添加到书签栏中，点击即可开始定时刷新当前页面。这种方式简单易用，非常适合需要长时间在线的测试工作。


# 技术实现


Bookmarklet 的核心是一段 Javascript 代码：


```javascript
let timeout = parseInt(prompt("Set timeout [s]"), 10);
const current = location.href;

if (timeout > 0) {
    setTimeout(reload, 1000 * timeout);
} 

function reload() {
    setTimeout(reload, 1000 * timeout);
    const iframeHTML = `<iframe src="${current}" style="width: 100%; height: 100%; border: none;"></iframe>`;
    document.open();
    document.write(iframeHTML);
    document.close();
}
```


它将当前页地址作为 iframe 重写到页面中，并设置定时器动态刷新


压缩成一行


```javascript
javascript:(function(){let timeout=parseInt(prompt("Set timeout [s]"),10);const current=location.href;if(timeout>0){setTimeout(reload,1000*timeout);}function reload(){setTimeout(reload,1000*timeout);const iframeHTML=`<iframe src="${current}" style="width: 100%; height: 100%; border: none;"></iframe>`;document.open();document.write(iframeHTML);document.close();}})();
```


这段代码利用 `setInterval` 方法每隔指定时间（例如5分钟）自动刷新当前页面。用户可以根据自己的需求调整 `interval` 的值，以适应不同的测试环境。




[https://codepen.io/z4none/pen/MYgJYpY](https://codepen.io/z4none/pen/MYgJYpY)


[embed](https://codepen.io/z4none/pen/MYgJYpY)

