---
updated: '2022-08-13 00:00:00'
categories: code
excerpt: 最近在装修博客页面过程中，想在页面上放类似 Github 项目上流行的 Badge，显示页面访问量。由于站点采用的 Nextjs + Vercel 的静态博客方案，一般来说需要使用第三方访问计数服务
date: '2022-08-13 00:00:00'
tags:
  - Web
urlname: cfworker-visitor-counter
title: 使用 Cloudflare Worker 实现访问统计
---

# 背景


最近在装修博客页面过程中，想在页面上放类似 Github 项目上流行的 Badge，显示页面访问量。由于站点采用的 Nextjs + Vercel 的静态博客方案，一般来说需要使用第三方访问计数服务，比如 [不蒜子](https://busuanzi.ibruce.info/) 。一番搜索，在 v2ex 上了解到了 [https://visitor-badge.glitch.me](https://visitor-badge.glitch.me/) 提供的服务，感觉比较有意思，通过返回 svg badge 图案显示访问数，使用起来也非常简单，直接显示一个指定地址的图片就行。


![Untitled.png](https://s.z4none.me/blog/0d2b50833ecd4baee06199a98d3a25ec.png)


其实现应该不复杂，有个 kv 数据库保存各页面的访问次数，请求时带上页面 id 从 kv 数据库中取出 +1 即可。刚好 Cloudflare 提供了 Worker 和 Worker KV 数据库，每天都有免费限额，快来白嫖。


# 实现

1. 先在 cf 后台创建一个名为 counter 的 kv 数据库
2. 创建一个新的 Worker
3. 在 Worker 的设置 / 变量 中将刚才的数据库绑定到变量 counter 上

![Untitled.png](https://s.z4none.me/blog/43f0ce8304c9e8aead1d54582432de40.png)

1. 在 Worker 快速编辑页面贴入代码：

```javascript
//
addEventListener("fetch", (event) => {
  event.respondWith(
    handleRequest(event.request).catch(
      (err) => new Response(err.stack, { status: 500 })
    )
  );
});

//
function generateBadge(subject, value, color) {
  const textLenght = (subject + value).length;
  const subjectLengt = ((subject.length + 4) / (textLenght + 8)) * 100;
  const valueLength = ((value.length + 4) / (textLenght + 8)) * 100;
  const fontSize = _getFontSize(textLenght);
  const borderRadius = 0;

  return generateBadgeSVG({
    borderRadius,
    subject,
    value,
    subjectLengt,
    valueLength,
    subjectColor: '#323B34',
    valueColor: color,
    fontSize
  });
}

//
function generateBadgeSVG({
  borderRadius = 4,
  fontSize = 9,
  height = 20,
  subject,
  subjectColor,
  subjectLengt,
  subjectTextColor = '#FFF',
  value,
  valueColor,
  valueLength,
  valueTextColor = '#FFF',
  width = 100,
}) {
  return `
  <svg style="border-radius: ${borderRadius}px" viewBox="0 0 ${width} ${height}" width="${width}" xmlns="http://www.w3.org/2000/svg">
    <g>
      <rect x="0" y="0" width="${subjectLengt}" height="${height}" fill="${subjectColor}" />
      <text font-size="${fontSize}px" font-weight="100" font-family="sans-serif" fill="${subjectTextColor}" x="${subjectLengt * 0.5}" y="55%" alignment-baseline="middle" text-anchor="middle">${subject}</text>
    </g>
    <g>
      <rect x="${subjectLengt}" y="0" width="${valueLength}" height="${height}" fill="${valueColor}" />
      <text font-size="${fontSize}px" font-weight="100" font-family="sans-serif" fill="${valueTextColor}"  x="${width - (valueLength / 2)}" y="55%" alignment-baseline="middle" text-anchor="middle">${value}</text>
    </g>
  </svg>
  `;
};

//
function _getFontSize(length) {
  if (length < 10) {
    return 12;
  }
  if (length < 13) {
    return 11;
  }
  if (length < 16) {
    return 10;
  }
  if (length < 20) {
    return 9;
  }
  if (length < 25) {
    return 8;
  }
  return 7;
}

//
async function handleRequest(request) {
  const { pathname } = new URL(request.url);
  const perfix = "/badge";

  if (pathname.startsWith(perfix)) {
    const path = pathname.substr(perfix.length);
    let count = +await counter.get(path);
    count += 1;
    counter.put(path, count)
    return new Response(generateBadge("visited", `${count}`, '#3b82f6'), {
      headers: { "Content-Type": "image/svg+xml" },
    });
  }

  if (pathname.startsWith("/status")) {
    const httpStatusCode = Number(pathname.split("/")[2]);

    return Number.isInteger(httpStatusCode)
      ? fetch("https://http.cat/" + httpStatusCode)
      : new Response("That's not a valid HTTP status code.");
  }

  const html = `<!DOCTYPE html>
<body>
  <h1>It works!</h1>
  <img src="/badge/hello"/>
</body>`;

  return new Response(html, {
    headers: {
      'content-type': 'text/html;charset=UTF-8',
    },
  });
}
```

1. 此时预览首页，可以看到测试计数的效果，每次刷新计数会 +1

![Untitled.png](https://s.z4none.me/blog/c1450d3c307c2c1fae83a1ab73b3ea95.png)

1. 使用方式：

	在要计数的页面上直接插入图片


	```javascript
	<img src="https://你的worker地址/badge/你的页面id"/>
	```


	```javascript
	![visitors](https://你的worker地址/badge/你的页面id)
	```


目前这个服务十分简单，只能记录页面的访问次数，并没有区分访客 UV，而且只能通过 cf 的后台浏览各个页面的访问次数，如果有访问统计的需求，还是得上 GA 或者 Ankee 这样的系统。

