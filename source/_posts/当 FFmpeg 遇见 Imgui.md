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


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/e0d557bf-02c8-49a5-8928-9bd9a3ea92ec/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T042913Z&X-Amz-Expires=3600&X-Amz-Signature=2b7635b3ac5c35ee67aa86d5684c542faab44dacd50358ba952c44ee0864d095&X-Amz-SignedHeaders=host&x-id=GetObject)


![2_%282%29.gif](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/8ea57dce-5287-4801-8406-c0a8b2c9a2dd/2_%282%29.gif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T042913Z&X-Amz-Expires=3600&X-Amz-Signature=1fc69877901880193efcbc9a277d4ae3e32096b12c69f940d895033edeac4d23&X-Amz-SignedHeaders=host&x-id=GetObject)

