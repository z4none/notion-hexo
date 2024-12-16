---
updated: '2014-07-04 00:00:00'
categories: code
excerpt: 想要对 list 分页的同时，还想知道当前页是否有 prev 和 next，类似如下操作：
date: '2014-07-04 00:00:00'
tags:
  - Python
urlname: python-paginator
title: Python list 分页
---

想要对 list 分页的同时，还想知道当前页是否有 prev 和 next，类似如下操作：


```text
paginator = Paginator(range(50), 12)
for page in paginator:
    page.has_prev # 是否有 prev
    page.has_next # 是否有 next
    page.index    # page 序号
    for item in page:
        item # 每个元素

```


不多说，上代码：


```text
# coding : utf-8
# file   : xpaginatior
# author : z4none@gmail.com

class Paginator:
    def __init__(self, items, page_size):
        self.items      = items[:]
        self.page_size  = page_size

    def __getitem__(self, index):
        return Page(self.items, self.page_size, index)
        return self.items[index * self.page_size: index * self.page_size + self.page_size]

    def __iter__(self):
        return Pages(self.items, self.page_size)

class Pages:
    def __init__(self, items, page_size):
        self.items      = items
        self.page_size  = page_size
        self.index      = 0

    def next(self):
        if self.index * self.page_size > len(self.items): raise StopIteration
        self.index = self.index + 1
        return Page(self.items, self.page_size, self.index-1)

class Page:
    def __init__(self, items, page_size, index):
        total = len(items)
        begin = index * page_size
        end   = index * page_size + page_size

        self.items      = items[begin: end]
        self.has_prev   = (begin > 0)
        self.has_next   = end < total
        self.index      = index

    def __iter__(self):
        return Items(self.items)

class Items:
    def __init__(self, items):
        self.items = items
        self.index = 0

    def next(self):
        if self.index >= len(self.items): raise StopIteration
        self.index = self.index + 1
        return self.items[self.index-1]

def test():
    paginator = Paginator(range(50), 12)
    for pages in paginator:
        print "[%02d]" % pages.index,
        print (pages.has_prev and "<" or " "),
        for item in pages:
            print "%02d" % item,
        print (pages.has_next and ">" or " "),
        print

if __name__ == "__main__":
    test()

```


以上代码输出结果：


```text
[00]   00 01 02 03 04 05 06 07 08 09 10 11 >
[01] < 12 13 14 15 16 17 18 19 20 21 22 23 >
[02] < 24 25 26 27 28 29 30 31 32 33 34 35 >
[03] < 36 37 38 39 40 41 42 43 44 45 46 47 >
[04] < 48 49
```

