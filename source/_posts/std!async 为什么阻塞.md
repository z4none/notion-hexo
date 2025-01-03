---
updated: '2022-08-07 00:00:00'
categories: code
excerpt: |-
  在桌面开发中，有时希望能减少某些 IO 操作对界面造成的影响，此前常用的办法是创建一个线程。这种方式创建了一个立即执行的分离线程，无需等待执行结束。
  在 C++ 11 标准中提供了 std::async, 对多线程异步操作进行了封装，简化了调用过程。
date: '2022-08-07 00:00:00'
tags:
  - C++
urlname: why-std-async-block
title: 'std::async 为什么阻塞'
---

在桌面开发中，有时希望能减少某些 IO 操作对界面造成的影响，此前常用的办法是创建一个线程：


```c++
std::thread([](){
	// do something
}).detach();
```


这种方式创建了一个立即执行的分离线程，无需等待执行结束。


在 C++ 11 标准中提供了 `std::async`, 对多线程异步操作进行了封装，简化了调用过程，并且还支持线程池。如果将刚才的分离线程改成用 `std::async` 实现，应该如下：


```c++
std::async(std::launch::async, [](){
	// do something
});
```


由于我们不关心返回值，所以似乎不需要获取返回的 `std::future`。但是在项目中却出现了 `std::async` 阻塞的情况，经过研究，下面是个最小复现的 demo：


![std::async 变成了同步调用](https://s.z4none.me/blog/7dfa2dc0800b34f27d812a18387ffc99.png)


 在 cppreference 上对这个问题是这么描述的：


> If the std::future obtained from std::async is not moved from or bound to a reference, the destructor of the std::future will block at the end of the full expression until the asynchronous operation completes, essentially making code such as the following synchronous:


如果从 std::async 返回的 std::future 没有绑定或者移动到其他引用，那么这个 std::future 的析构函数将会阻塞直到整个异步操作完成，这将使得 `std::async` 像是同步一样。


所以，要想让 `std::async` 真正异步，需要将其返回值保存到变量中，确保其在需要的时候再析构或者调用 `.get` 。

