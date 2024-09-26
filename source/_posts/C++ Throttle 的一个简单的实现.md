---
updated: '2022-11-19 08:00:00'
categories: code
excerpt: 说到 Throttle，在网页前端中运用得较多，比如页面滚动、即时搜索的输入事件，这类事件触发非常频繁，如果每次都进行业务操作，消耗会非常大。这时采用 Throttle 进行限流，当函数执行后一段时间内不重复执行，俗称冷却。
date: '2022-11-19 08:00:00'
tags:
  - C++
urlname: cxx-throttle
title: C++ Throttle 的一个简单的实现
---

说到 Throttle，在网页前端中运用得较多，比如页面滚动、即时搜索的输入事件，这类事件触发非常频繁，如果每次都进行业务操作，消耗会非常大。这时采用 Throttle 进行限流，当函数执行后一段时间内不重复执行，俗称冷却。在 C++ 开发中，有时也会遇到相似的场景：在一个不停执行的循环中需要定时进行某些操作，常规的处理方式是类似这样：


```c++
time_t begin = time(0);
while(!stop)
{
	if(time(0) - begin > 5)
	{
		begin = time(0);
		doSomething();
	}

	doOtherBiz();
}
```


看起来简单明了的解决了问题，现在如果再来一个需要定时执行的操作，那么代码需要改成这样：


```c++
time_t begin1 = time(0);
time_t begin2 = time(0);
while(!stop)
{
	if(time(0) - begin1 > 5)
	{
		begin1 = time(0);
		doSomething1();
	}

	if(time(0) - begin2 > 10)
	{
		begin2 = time(0);
		doSomething2();
	}

	doOtherBiz();
}
```


看起来不太妙，让我们把这种处理方式简单封装一下：


```c++
#include <chrono>

using std::chrono::duration_cast;
using std::chrono::milliseconds;
using std::chrono::system_clock;

class Throttle
{
    int64_t m_lastTm = 0;
    int64_t m_interval = 0;
    std::function<void()> m_func = nullptr;

    int64_t now()
    {
        return duration_cast<milliseconds>(system_clock::now().time_since_epoch()).count();
    }
public:
    Throttle(int64_t interval, std::function<void()> func)
    {
        m_interval = interval;
        m_func = func;
    }

    void operator () (void) 
    {
        if (m_lastTm + m_interval <= now())
        {
            m_lastTm = now();
            m_func();
        }
    }
};
```


可以这么使用，是不是方便了很多呢？


```c++
Throttle doSomething1(5000, [&]() {
    ...
});

Throttle doSomething2(10000, [&]() {
    ...
});

while(!stop)
{
	doSomething1();
	doSomething2();
	doOtherBiz();
}
```


 与 `Javascript` 用定时器实现的 Throttle 不同的是，跳过执行的调用在冷却过去后并不会自动执行，不过在高频运行的循环中这点问题可以忽略不计了。

