---
updated: '2020-12-17 00:00:00'
categories: code
excerpt: |-
  为了自定义 pyQT5 项目图标, 有三处需要替换，分别为：
  • 窗口图标
  • 任务栏图标
  • 应用程序图标
  下面分别进行介绍。
date: '2020-12-17 00:00:00'
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


![Untitled.png](https://s.z4none.me/blog/8a00c4a708d8e88c7b1af85baf521def.png)


## 窗口图标


采用 `setWindowIcon` 来设置应用程序窗口图标


```text
setWindowIcon(QtGui.QIcon(‘icon.png’))
```


可以设置到指定窗口上, 也可设置到 app 对象上, 区别是设置到 app 对象上时如果窗口没有设置图标,则取 app 图标作为默认([https://doc.qt.io/qt-5/qwidget.html#windowIcon-prop](https://doc.qt.io/qt-5/qwidget.html#windowIcon-prop)).


设置完成后效果如下


![Untitled.png](https://s.z4none.me/blog/63551056e002e8d3d0b219bb85e4aa44.png)


## 任务栏图标


在 Windows 平台下, 任务栏图标是跟随 application 图标的, 但是对于 Python 项目而言情况略微复杂.


我们知道 Python 是脚本语言, 其代码执行需要 Python Interpreter, 所以当以脚本方式执行代码时, 真正运行的是 Python Interpreter, 对应的图标也是 Python Interpreter 的图标.


不过就算如此, 我们也能对其进行修改. 在 Windows 平台下, 任务栏图标是与 Application User Models 关联的, 而 Application User Models 的唯一标识是 [AppUserModelIDs](https://docs.microsoft.com/zh-cn/windows/win32/shell/appids?redirectedfrom=MSDN#host). 我们可以在程序运行时修改 AppUserModelIDs, 使其于 Pythonw.exe 解除关联, 从而显示其自身的图标.


```text
import ctypes
myappid = 'mycompany.myproduct.subproduct.version'  # arbitrary string
ctypes.windll.shell32.SetCurrentProcessExplicitAppUserModelID(myappid)
```


![Untitled.png](https://s.z4none.me/blog/4df14406149a2fec3232b7562eca836d.png)


[https://stackoverflow.com/a/1552105](https://stackoverflow.com/a/1552105)


如果项目采用 PyInstaller 等打包工具编译成 Exe, 可以通过相关参数指定应用程序图标, 参见下一节.


## 应用程序图标


Pyinstaller 编译命令行


```text
pyinstaller.exe --onefile --windowed --icon=cat.ico main.py  -y

```


编译后效果如下


![Untitled.png](https://s.z4none.me/blog/181bdbadc1c27643244a2f573769bba1.png)

