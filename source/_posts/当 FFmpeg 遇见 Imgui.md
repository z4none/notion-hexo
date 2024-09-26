---
updated: '2020-05-31 08:00:00'
categories: code
excerpt: Dear Imgui 是一个立即模式的 C++ GUI lLibrary，与其他 GUI 框架不同，它将渲染与框架逻辑分离，用户可以根据自己的需要选择渲染方式，官方支持如下
date: '2020-05-31 08:00:00'
tags:
  - C++
urlname: ffmpeg-meet-imgui
title: 当 FFmpeg 遇见 Imgui
---

Dear Imgui 是一个立即模式的 C++ GUI lLibrary，与其他 GUI 框架不同，它将渲染与框架逻辑分离，用户可以根据自己的需要选择渲染方式，官方支持如下：


```text
Renderers: DirectX9, DirectX10, DirectX11, DirectX12, OpenGL (legacy), OpenGL3/ES/ES2 (modern), Vulkan, Metal.
Platforms: GLFW, SDL2, Win32, Glut, OSX.
```


有时我们需要在 FFMpeg 应用中使用 GUI，传统的方式是将 FFMpeg 嵌入到 GUI 应用程序中， 有了 Imgui，我们可以在视频画面之上叠加我们的 GUI 界面。


![Untitled.png](https://s.z4none.me/blog/4cd8d719226a7f8cc875e832ff223e57.png)


![2_%282%29.gif](https://s.z4none.me/blog/93104346773e922a9477c00ba5aa8f26.gif)

