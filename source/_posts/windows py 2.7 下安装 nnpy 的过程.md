---
updated: '2017-12-21 00:00:00'
categories: code
excerpt: nnpy 是 nanomsg 的 python bindings， 在持续更新中， 作者似乎不太关注 windows 平台， 但是 nanomsg 和 cffi 都是支持 windows 的， 所以 nnpy 也可以安装到 windows 的 python 中， 只是直接安装会报错， 下面记录一下安装的过程。
date: '2017-12-21 00:00:00'
tags:
  - C++
  - Python
urlname: windows-nnpy
title: windows py 2.7 下安装 nnpy 的过程
---

nnpy 是 nanomsg 的 python bindings， 在持续更新中， 作者似乎不太关注 windows 平台， 但是 nanomsg 和 cffi 都是支持 windows 的， 所以 nnpy 也可以安装到 windows 的 python 中， 只是直接安装会报错， 下面记录一下安装的过程。


首先相关环境如下： > win10 64 位 python 2.7.13 / win32


# 编译 nanomsg


先下载 [http://nanomsg.org/download.html](http://nanomsg.org/download.html) 我下载的是 1.1.2


编译需要用 CMake 生成 VC 工程， 所以需要先安装好 CMake 和 VC


执行 build_all 和 install, 然后在 install 路径下可以看到有编译好的 lib / bin 以及 include


# 安装 nnpy


先下载 [https://github.com/nanomsg/nnpy/releases](https://github.com/nanomsg/nnpy/releases) 我下载的是 1.4.1

- 将上一步的头文件复制到 `C:\Python27\include` 文件夹中，并将 nnpy 的 generate.py 中的 `DEFAULT_INCLUDE_DIRS` 的值改为 `[r'C:\Python27\include\nanomsg']`
- 将上一步的 nanomsg.dll 复制到 `C:\Python27\DLLs` 中，并将 nnpy 的 generate.py 中的 `DEFAULT_HOST_LIBRARY` 的值改为 `r'C:\Python27\DLLs\nanomsg.dll'`
- 在 nnpy 目录中执行 `pip install .` （可能需要管理员权限？），如果顺利应该可以安装成功
- 此时在 python 中 import nnpy 应该会报错 “找不到模块”，可以把 `C:\Python27\Lib\site-packages` 中的 _nnpy.pyd 移动到 `C:\Python27\DLLs` （和 nanomsg.dll 一起）来解决
