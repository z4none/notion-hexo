---
updated: '2013-07-05 08:00:00'
categories: code
excerpt: 在 Python 多线程中使用 print 有时会将多行内容输出到一行，这是因为 Python 是将要输出的内容和换行符分开打印的。 要避免这个问题，方法之一就是对 print 做线程同步操作
date: '2013-07-05 08:00:00'
tags:
  - Python
urlname: python-multi-thread-print
title: 在 Python 多线程中使用 print
---

在 Python 多线程中使用 print 有时会将多行内容输出到一行，这是因为 Python 是将要输出的内容和换行符分开打印的。 要避免这个问题，方法之一就是对 print 做线程同步操作：


```text
def sync_print(message, lock=threading.Lock()):
    with lock:
        print message

```


这里用了 Python 对函数参数的默认值只会初始化一次的特点省去了创建全局 Lock 的麻烦。 如果采用 Python 3.0 的方式，可以将 print 函数改成 sys.stdout.write 实现无缝替换：


```text
from __future__ import print_function
print = lambda x: sys.stdout.write("%s\n" % x)
```

