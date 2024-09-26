---
updated: '2020-12-17 08:00:00'
categories: code
excerpt: |-
  为了自定义 pyQT5 项目图标, 有三处需要替换，分别为：
  • 窗口图标
  • 任务栏图标
  • 应用程序图标
  下面分别进行介绍。
date: '2020-12-17 08:00:00'
tags:
  - C++
urlname: pyqt-app-icon
title: 如何设置 pyQT5 程序图标
---

为了自定义 pyQT5 项目图标, 有三处需要替换，分别为：

- 窗口图标
- 任务栏图标
- 应用程序图标

下面分别进行介绍。


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/43c677bf-f0bb-4358-b300-59a95e8033b3/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T043409Z&X-Amz-Expires=3600&X-Amz-Signature=d3aa24232e6cc5893b2e66fd21bf393b721a84b7be82a10587f90a79eaf02dd9&X-Amz-SignedHeaders=host&x-id=GetObject)


## 窗口图标


采用 `setWindowIcon` 来设置应用程序窗口图标


```text
setWindowIcon(QtGui.QIcon(‘icon.png’))
```


可以设置到指定窗口上, 也可设置到 app 对象上, 区别是设置到 app 对象上时如果窗口没有设置图标,则取 app 图标作为默认([https://doc.qt.io/qt-5/qwidget.html#windowIcon-prop](https://doc.qt.io/qt-5/qwidget.html#windowIcon-prop)).


设置完成后效果如下


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/56ffb263-584a-408b-9006-23cffd3ccf7e/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T043408Z&X-Amz-Expires=3600&X-Amz-Signature=ef1d99e0077f3866f22e815263caca1be6e434839e67ee40b9ccfa986902b84e&X-Amz-SignedHeaders=host&x-id=GetObject)


## 任务栏图标


在 Windows 平台下, 任务栏图标是跟随 application 图标的, 但是对于 Python 项目而言情况略微复杂.


我们知道 Python 是脚本语言, 其代码执行需要 Python Interpreter, 所以当以脚本方式执行代码时, 真正运行的是 Python Interpreter, 对应的图标也是 Python Interpreter 的图标.


不过就算如此, 我们也能对其进行修改. 在 Windows 平台下, 任务栏图标是与 Application User Models 关联的, 而 Application User Models 的唯一标识是 [AppUserModelIDs](https://docs.microsoft.com/zh-cn/windows/win32/shell/appids?redirectedfrom=MSDN#host). 我们可以在程序运行时修改 AppUserModelIDs, 使其于 Pythonw.exe 解除关联, 从而显示其自身的图标.


```text
import ctypes
myappid = 'mycompany.myproduct.subproduct.version'  # arbitrary string
ctypes.windll.shell32.SetCurrentProcessExplicitAppUserModelID(myappid)
```


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/96fa958d-a7f7-4919-804a-17923768ecb2/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T043409Z&X-Amz-Expires=3600&X-Amz-Signature=f7a6d5d41b19e60a08aa07f295f1e51afef0e0c80c17c9d412f4a0ddacd3e354&X-Amz-SignedHeaders=host&x-id=GetObject)


[https://stackoverflow.com/a/1552105](https://stackoverflow.com/a/1552105)


如果项目采用 PyInstaller 等打包工具编译成 Exe, 可以通过相关参数指定应用程序图标, 参见下一节.


## 应用程序图标


Pyinstaller 编译命令行


```text
pyinstaller.exe --onefile --windowed --icon=cat.ico main.py  -y

```


编译后效果如下


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/b0b15290-470e-4769-895b-02ea12a10afb/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T043409Z&X-Amz-Expires=3600&X-Amz-Signature=456408f722dfdeabefba3dac52ac1690ef25708b46452938f513e32c0a085870&X-Amz-SignedHeaders=host&x-id=GetObject)

