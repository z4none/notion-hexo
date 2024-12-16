---
updated: '2019-12-02 00:00:00'
categories: code
excerpt: uTools 可以自动搜索开始菜单中安装好的程序，对于绿色软件，可以将其快捷方式保存在一个文件夹中并且在插件偏好设置中，设置 “自定义快捷方式目录”，这个操作在 ALMRun 中已集成到 Windows 的右键菜单的发送项中，在 uTools 中就没有那么方便了，只能靠手动完成。
date: '2019-12-02 00:00:00'
tags:
  - Windows
urlname: win10-send-shortcut-to-directory
title: Win10 发送快捷方式到指定文件夹
---

在 Windows 平台下有不少快速启动工具，可以通过快捷键呼出，然后通过关键字启动指定软件。我之前使用的是 [ALMRun](https://github.com/chenall/ALMRun)，最近发现了另一个工具 [uTools](https://u.tools/)。


实际上 uTool 并不单是一个启动器，而是一组工具集，并且可以通过插件对其功能进行扩展。通过插件一些常用的功能就不需要打开另外的软件完成了，比如屏幕取色、计算文件 MD5、文本翻译等等。


本文讨论的是 uTool 作为程序启动器的使用相关配置。


uTools 可以自动搜索开始菜单中安装好的程序，对于绿色软件，可以将其快捷方式保存在一个文件夹中并且在插件偏好设置中，设置 “自定义快捷方式目录”，这个操作在 ALMRun 中已集成到 Windows 的右键菜单的发送项中，在 uTools 中就没有那么方便了，只能靠手动完成。


于是我写了一个批处理脚本来简化创建快捷方式的操作：


```text
@echo off
title=z4none.me
set Program=%~1
set /p LinkName=shortcut name:
set WorkDir=%~dp1
set WorkDir=%WorkDir:~,-1%
(echo Set WshShell=CreateObject("WScript.Shell"^)
echo strDest="C:\\Tools\\uTools\\shortcuts\\"
echo Set oShellLink=WshShell.CreateShortcut(strDest^&"\%LinkName%.lnk"^)
echo oShellLink.TargetPath="%Program%"
echo oShellLink.WorkingDirectory="%WorkDir%"
echo oShellLink.WindowStyle=1
echo oShellLink.Description="utools shortcut"
echo oShellLink.Save)>%temp%\\makelnk.vbs
echo done!
%temp%\\makelnk.vbs
del /f /q %temp%\\makelnk.vbs
exit

```


其中 strDest 为固定路径，表示你存放快捷方式的文件夹，可根据需要修改。


将这个批处理文件保存到 Windows 的 Sendto 文件夹，打开方式为：在我的电脑中访问 shell:sendto，批处理可命名为你想要的名称


保存完毕后可以在右键菜单中即可看到右键的发送中出现了此菜单项 (加号开头可排在最前)


![Untitled.png](https://s.z4none.me/blog/5fffcc00e3b82c2d645e782ad09cef5e.png)


选择此项后会出现提示符要求输入快捷方式的名称（用于搜索的名称），回车确认后快捷方式即可创建成功


![Untitled.png](https://s.z4none.me/blog/be11877843ab3263c85c5588043d6c88.png)


确保设置了 uTools 的自定义快捷方式目录，在搜索框中输入该名称即可查到对应的程序。


![Untitled.png](https://s.z4none.me/blog/916f86cc41a0f601e69d0ef30f710619.png)

