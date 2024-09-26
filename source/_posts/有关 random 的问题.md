---
updated: '2018-08-05 08:00:00'
categories: code
excerpt: "在开发工程中如果需要生成随机数, 一般是通过 rand 函数, 它可以生成 0 到 RAND_MAX 范围的一个\_伪随机数, 如果要让 rand 每次生成的随机序列不同, 可以通过 srand 函数不同的种子值, 一般设定为当前时间 srand(time(NULL)).\n以上是我之前对随机数的基本理解, 今天我在编码工程中发现了一些新的随机数相关的问题, 让我对随机数有了进一步的了解, 下面通过几段代码来进行说明."
date: '2018-08-05 08:00:00'
tags:
  - C++
urlname: random
title: 有关 random 的问题
---

在开发工程中如果需要生成随机数, 一般是通过 rand 函数, 它可以生成 0 到 RAND_MAX 范围的一个 `伪随机数`, 如果要让 rand 每次生成的随机序列不同, 可以通过 srand 函数不同的种子值, 一般设定为当前时间 srand(time(NULL)).


以上是我之前对随机数的基本理解, 今天我在编码工程中发现了一些新的随机数相关的问题, 让我对随机数有了进一步的了解, 下面通过几段代码来进行说明.


## srand 对 rand 的影响


Windows VS 2017 输出


```text
for (int i = 0; i < 10; i++)
{
    srand(i);
    printf("seed = %d, rand = %d\n", i, rand());
}

// 输出
seed = 0, rand = 38
seed = 1, rand = 41
seed = 2, rand = 45
seed = 3, rand = 48
seed = 4, rand = 51
seed = 5, rand = 54
seed = 6, rand = 58
seed = 7, rand = 61
seed = 8, rand = 64
seed = 9, rand = 68

```


可以看出在 Windows 平台 VC 下, 随着 srand seed 参数的增长, rand 首次的值也是增长的, 所以如果在 VC 下写以下这样的代码:


```text
int rand_number()
{
    srand(time(NULL));
    return rand();
}

for(int i=0; i<10; i++)
{
    printf("%d\n", rand_number());
    Sleep(1000);
}

```


那么会发现它每次生成的随机数是递增的


windows 下 srand 和 rand 函数的实现如下:


```text
void __cdecl srand (unsigned int seed)
{
    #ifdef _MT
        _getptd()->_holdrand = (unsigned long)seed;
    #else /* _MT */
        holdrand = (long)seed;
    #endif /* _MT */
}

int __cdecl rand (void)
{
    #ifdef _MT
        _ptiddata ptd = _getptd();
        return( ((ptd->_holdrand = ptd->_holdrand * 214013L + 2531011L) >> 16) & 0x7fff );
    #else /* _MT */
        return(((holdrand = holdrand * 214013L + 2531011L) >> 16) & 0x7fff);
    #endif /* _MT */
}

```


可以看到 ptd->_holdrand 和 rand 返回值的确是线性关系(线性同余法), 所以每次设定 srand 的 seed 值后的首次 rand 是随 seed 增加而增加的.


同样的代码在 Ubuntu 下


```text
for (int i = 0; i < 10; i++)
{
    srand(i);
    printf("seed = %d, rand = %d\n", i, rand());
}

// 输出
seed = 0, rand = 1804289383
seed = 1, rand = 1804289383
seed = 2, rand = 1505335290
seed = 3, rand = 1205554746
seed = 4, rand = 1968078301
seed = 5, rand = 590011675
seed = 6, rand = 290852541
seed = 7, rand = 1045618677
seed = 8, rand = 757547896
seed = 9, rand = 444454915

```


## 多线程


此问题同样需要分平台讨论, 在 Windows 平台下, srand 的 seed 是关联到线程的(参见前文代码), 也就是说需要在各个线程分别调用 srand.


如果我们有一组线程同时创建, 那么 srand(time(NULL)) 很有可能获得相同的 seed 值, 这种情况一般是根据 time(NULL) 和 thread_id 或者局部变量地址组合生成 seed 用于初始化 srand:


```text
//
void thread_func(int foo)
{
    srand(time(NULL) * int(&foo));
    printf("rand = %d\n", rand());
}

```


在 Linux 平台下, 只需在主线程中调用 srand 即可, 不过由于 rand 使用了内部的隐藏状态, 所以它不保证在多线程环境下行为可复现, 如果需要实现可复现的 rand 序列, 需要采用 rand_r() 函数.


## 生成范围内的随机数


在实际应用中往往会需要生成一定范围内的随机数, 较简单实现方式如下:


```text
int range(int from, int to)
{
    return from + rand() % (to - from + 1);
}

```


当对数据要求不高时, 以上算法可以满足要求, 它有几个问题: 1. 数据范围不能超过 RAND_MAX 2. 数据分布不平均


为了解决数据范围的问题, 可以采用以下方式扩大生成的随机数范围:


```text
int rand1()
{
    return (rand() * RAND_MAX) | rand();
}

```


为了解决数据分布问题, 可采用以下方法:


```text
int range(int from, int to)
{
    return from + double(rand()) * (to - from) / RAND_MAX;
}

```


不过即使是这样, 也不能保证完全平均, 随机到 to 值的几率也只有 1 / RAND_MAX.


## 使用 C++ 11 的随机数生成器


现在推荐使用 C++ 11 的随机数生成器生成平均分布的任意范围的随机数, 使用方法如下:


```text
std::random_device rd;
std::mt19937 mt(rd());
std::uniform_int_distribution<int> dist(1, 100);

for (int i = 0; i < 10; i++)
{
	printf("main, rand = %d\n", dist(mt));
}
```

