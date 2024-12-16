---
updated: '2023-11-20 00:00:00'
categories: code
excerpt: '在 Windows 桌面程序开发中，获取 int argc, char const *argv[] 的方法'
date: '2023-11-20 00:00:00'
tags:
  - C++
  - Windows
urlname: get-argv-argc-in-winmain
title: '在 WinMain 中获取 argv, argc'
---

在 Windows 桌面程序开发中，一般的程序入口是


```c++
int APIENTRY _tWinMain(_In_ HINSTANCE hInstance,
    _In_opt_ HINSTANCE hPrevInstance,
    _In_ LPTSTR    lpCmdLine,
    _In_ int       nCmdShow)
```


有时需要获取 `int argc, char const *argv[]` ，可以使用[ __wargv, __argv, __argc 全局变量](https://learn.microsoft.com/en-us/cpp/c-runtime-library/argc-argv-wargv?view=msvc-170)。在 VC 环境下，__argc 是命令行参数个数，__argv 是命令行参数指针数组（字符为 multibyte 格式），__wargv 是wide-character 格式的命令行参数指针数组。要注意 __argv  或 __wargv 的值与项目字符集设置相关，比如项目设置为 Unicode, 那么 __argv 的值为空。


为了避免此问题，可以使用以下代码：


```c++
void GetCommandLineArgs(int* argc, char*** argv)
{
    // 获取 wchar_t, 命令行参数
    wchar_t** wargv = CommandLineToArgvW(GetCommandLineW(), argc);
    if (!wargv) { *argc = 0; *argv = NULL; return; }

    // 计算所需内存空间
    int n = 0;
    for (int i = 0; i < *argc; i++)
        n += WideCharToMultiByte(CP_UTF8, 0, wargv[i], -1, NULL, 0, NULL, NULL) + 1;

    // 分配内存：指针所需空间 + 字符串所需空间
    *argv = (char **)malloc((*argc + 1) * sizeof(char*) + n);
    if (!*argv) { *argc = 0; return; }

    // 转换为 u8
    char* arg = (char*)&((*argv)[*argc + 1]);
    for (int i = 0; i < *argc; i++)
    {
        (*argv)[i] = arg;
        arg += WideCharToMultiByte(CP_UTF8, 0, wargv[i], -1, arg, n, NULL, NULL) + 1;
    }
    (*argv)[*argc] = NULL;
		// argv 存储格式为：参数指针 + 参数字符串 + NULL
}

```

