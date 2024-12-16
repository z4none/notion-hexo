---
updated: '2024-11-20 00:00:00'
categories: code
excerpt: FFmpeg 7.0 在 Windows 7 系统上出现访问错误，原因是其依赖的某些系统函数在 Windows 7 中未实现。解决方案是自行编译所需的 DLL 文件并实现相关功能。
date: '2024-11-20 00:00:00'
tags:
  - C++
urlname: ffmpeg-windows7-access-violation
title: FFmpeg 7 在 Windows 7 上不能运行的问题
---

# 问题


我们的项目中 FFmpeg 升级到了版本 7.0，结果有用户反馈在 Windows7 系统上出现访问错误（0xc0000005）。


![image.png](https://s.z4none.me/blog/9b63560d728b2921f35b67fb368e791e.png)


经过简单分析，发现 FFmpeg 中的 `avcodec-61.dll` 依赖了 `API-MS-WIN-CORE-SYNCH-L1-2-0.DLL` 


这个文件中的 3 个导出函数实现缺失导致出了问题。


![991d96d77946028660dbeeda29cbac9a.png](https://s.z4none.me/blog/6cb36469a0c296f8e8e961cd4a642c6a.png)


`API-MS-WIN` 开头的这些文件是在 Windows8 引入的，目的是为了提升程序的兼容性，上面的这个 dll 实际上是将导出函数重定向到了 kernel32.dll 中了，但是这 3 个函数: `WakeByAddressAll`, `WakeByAddressSingle` 和 `WakeOnAddress`  在 Windows 7 的 kernel32.dll 中是没有实现的。


![image.png](https://s.z4none.me/blog/e65d9571e8b2589329438230d80982d7.png)


# 解决方法


方法 1：裁剪 FFmpeg，移除使用这些 API 的功能。


暂时不考虑此方案。


方法 2：解决函数定义缺失问题


我们可以自己编译一个 `API-MS-WIN-CORE-SYNCH-L1-2-0.DLL`，将其放置在 `avcodec-61.dll` 同目录下并实现相关功能。在 Github 上，我找到了一个专门解决此问题的项目（原本用于使 Qt Creator 能在 Win7 上运行）。


[https://github.com/z4none/api-ms-win-core-synch-Win7](https://github.com/z4none/api-ms-win-core-synch-Win7)


参考资料


[https://www.reddit.com/r/ffmpeg/comments/1ejm2e8/ffmpeg_not_running_on_windows_7/](https://www.reddit.com/r/ffmpeg/comments/1ejm2e8/ffmpeg_not_running_on_windows_7/)


[https://github.com/GyanD/codexffmpeg/issues/136](https://github.com/GyanD/codexffmpeg/issues/136)

