---
updated: '2020-12-14 08:00:00'
categories: code
excerpt: 当开发 Windows 桌面应用程序时，我们时常需要对程序的崩溃信息进行分析，Windows 提供了 minidump 机制能将程序崩溃状态保存下来进行分析，前提是需要结合对应版本的 pdb 和 exe 文件。
date: '2020-12-14 08:00:00'
tags:
  - C++
urlname: modify-symbol-file
title: 修改 symbol 文件 signature
---

# 问题


当开发 Windows 桌面应用程序时，我们时常需要对程序的崩溃信息进行分析，Windows 提供了 minidump 机制能将程序崩溃状态保存下来进行分析，前提是需要结合对应版本的 pdb 和 exe 文件。


比如如下程序：


```text
//
#include <iostream>int main()
{
    std::cout << "Hello World!\n";

    int* p = 0;
    *p = 0;

    std::cout << "Fatal Error\n";
    return 0;
}

```


运行时会发生访问异常，配合 `MiniDumpWriteDump` API 会生成异常发生时的 dmp 文件，配合 exe 和 pdb 文件，再 WinDbg 中即可看到崩溃发生的位置，这对于我们排查问题十分有帮助。


```text
EXCEPTION_RECORD:  (.exr -1)
ExceptionAddress: 00007ff7e8b57919 (test_symbol!main+0x0000000000000049)
   ExceptionCode: c0000005 (Access violation)
  ExceptionFlags: 00000000
NumberParameters: 2
   Parameter[0]: 0000000000000001
   Parameter[1]: 0000000000000000
Attempt to write to address 0000000000000000

WRONG_SYMBOLS_TIMESTAMP: 70e69bad

WRONG_SYMBOLS_SIZE: 1f5000

FAULTING_MODULE: 00007fff5da90000 ntdll

ADDITIONAL_DEBUG_TEXT:
You can run '.symfix; .reload' to try to fix the symbol path and load symbols. ; Followup set based on attribute [Is_ChosenCrashFollowupThread] from Frame:[0] on thread:[PSEUDO_THREAD]

STACK_TEXT:
00000045`3f7bf820 00007ff7`e8b57919 test_symbol!main+0x49
00000045`3f7bf940 00007ff7`e8b58cc9 test_symbol!invoke_main+0x39
00000045`3f7bf990 00007ff7`e8b58bae test_symbol!__scrt_common_main_seh+0x12e
00000045`3f7bfa00 00007ff7`e8b58a6e test_symbol!__scrt_common_main+0xe
00000045`3f7bfa30 00007ff7`e8b58d59 test_symbol!mainCRTStartup+0x9
00000045`3f7bfa60 00007fff`5c856fd4 kernel32!BaseThreadInitThunk+0x14
00000045`3f7bfa90 00007fff`5dadcec1 ntdll!RtlUserThreadStart+0x21


STACK_COMMAND:  .ecxr ; kb ; ** Pseudo Context ** Pseudo ** Value: 2094f0d9800 ** ; kb

FAULTING_SOURCE_LINE:  F:\workspace\test\test_symbol\test_symbol.cpp

FAULTING_SOURCE_FILE:  F:\workspace\test\test_symbol\test_symbol.cpp

FAULTING_SOURCE_LINE_NUMBER:  9

FAULTING_SOURCE_CODE:
     5: {
     6:     std::cout << "Hello World!\n";
     7:
     8:     int* p = 0;
>    9:     *p = 0;
    10:
    11:     std::cout << "Fatal Error\n";
    12:     return 0;
    13: }


BUGCHECK_CODE:  70e69bad

EXCEPTION_CODE_STR:  70E69BAD

```


> dmp 文件可以通过 CrashDumper 辅助类自动生成


	[https://gist.github.com/z4none/502622698d5eda0fb69bd71737724962](https://gist.github.com/z4none/502622698d5eda0fb69bd71737724962)


在实际的开发工作中，软件版本是不停的迭代的，为了对不同版本的 exe 和 pdb 文件进行统一管理，微软提供了 symstore.exe 来将 exe 和 pdb 收录到符号服务器上 Symbol Server。


> 每次发布新版本软件时，通过 symstore add 命令将 exe 和 pdb 保存到指定目录，比如


	symstore add /r /f .*.* /s c:\mysymbol /t test


WinDbg 或者 cdb 能够通过配置符号服务器自动加载正确的 exe 和 pdb 进行分析。


> 在环境变量中配置 _NT_SYMBOL_PATH = c:\mysymbol 即可配置 WinDbg 可用的自定义 Symbol Server 或者目录


但是有时我们发布程序时，有时会忘记了保存符号文件，当有 dmp 需要分析时已经太晚了，即使项目 checkout 到对应版本重新编译得到了 exe 和 pdb，WinDbg 也会抱怨说：WRONG_SYMBOLS_TIMESTAMP，此时只能勉强看到出错的模块和大致的堆栈，十分不便。


```text
EXCEPTION_RECORD:  (.exr -1)
ExceptionAddress: 00007ff700837919 (test_symbol+0x0000000000017919)
   ExceptionCode: c0000005 (Access violation)
  ExceptionFlags: 00000000
NumberParameters: 2
   Parameter[0]: 0000000000000001
   Parameter[1]: 0000000000000000
Attempt to write to address 0000000000000000

WRONG_SYMBOLS_TIMESTAMP: 70e69bad

WRONG_SYMBOLS_SIZE: 1f5000

FAULTING_MODULE: 00007fff5da90000 ntdll

ADDITIONAL_DEBUG_TEXT:
You can run '.symfix; .reload' to try to fix the symbol path and load symbols. ; Followup set based on attribute [Is_ChosenCrashFollowupThread] from Frame:[0] on thread:[PSEUDO_THREAD]

STACK_TEXT:
00000013`426ff7e0 00007ff7`00837919 test_symbol+0x17919
00000013`426ff7e8 00007fff`07af7110 MSVCP140D!std::cout+0x0
00000013`426ff7f0 00007ff7`00842638 test_symbol+0x22638
00000013`426ff7f8 cccccccc`cccccccc unknown!unknown+0x0
00000013`426ff800 cccccccc`cccccccc unknown!unknown+0x0
00000013`426ff808 cccccccc`cccccccc unknown!unknown+0x0


STACK_COMMAND:  .ecxr ; kb ; ** Pseudo Context ** Pseudo ** Value: 210e988d7e0 ** ; kb

BUGCHECK_CODE:  70e69bad

EXCEPTION_CODE_STR:  70E69BAD

EXCEPTION_STR:  WRONG_SYMBOLS

PROCESS_NAME:  ntdll.wrong.symbols.dll

IMAGE_NAME:  ntdll.wrong.symbols.dll

MODULE_NAME: ntdll_wrong_symbols

SYMBOL_NAME:  ntdll_wrong_symbols!70E69BAD1F5000

FAILURE_BUCKET_ID:  WRONG_SYMBOLS_X64_19041.1.amd64fre.vb_release.191206-1406_TIMESTAMP_300109-035525_70E69BAD_ntdll.wrong.symbols.dll!70E69BAD1F5000

```


# 原因


要解决这个问题，需要从 symstore 备份 exe 和 pdb 文件的方式说起。


symstore.exe 每次备份时通过 exe 的 timestamp 和 pdb 的 signature 等信息将其存入指定路径，这样就能保证多个版本的同名文件可以共存于一个 Symbol Server 上：


exe 的存储路径为


```text
test_symbol.exe/5FD78A8030000/test_symbol.exe

```


其中 5FD78A80 为 Timestamp，30000 为 image size


pdb 的存储路径为


```text
test_symbol.pdb/B0FFD19C07A44043A4F615DD7BE108021/test_symbol.pdb

```


其中 B0FFD19C07A44043A4F615DD7BE10802 为 Signature，1 为 age


VisualStudio 每次编译程序时会自动修改文件的 Timestamp 和 Signature，所以即使是相同的代码编译出的 exe 和 pdb，和旧版本的 dmp 文件也无法匹配。


## 解决方法


既然 VisualStudio 每次会生成新的 Timestamp 和 Signature，那么为了能让 dmp 文件找到 exe 和 pdb， 那我们就需要将 timestamp 和 Signature 改回对应的值。


在 WinDbg 先执行 `!sym noisy` 命令，然后再执行`!analyze -v` 可以看到 WinDbg 的查找过程，且其中就包含 Timestamp 和 Signature


```text
SYMSRV:  BYINDEX: 0x9
         c:\winsymbols
         test_symbol.exe
         5FDA21D230000
SYMSRV:  UNC: c:\winsymbols\test_symbol.exe\5FDA21D230000\test_symbol.exe - path not found
SYMSRV:  UNC: c:\winsymbols\test_symbol.exe\5FDA21D230000\test_symbol.ex_ - path not found
SYMSRV:  UNC: c:\winsymbols\test_symbol.exe\5FDA21D230000\file.ptr - path not found
SYMSRV:  RESULT: 0x80070003
SYMSRV:  BYINDEX: 0xA
         c:\winsymbols*http://msdl.microsoft.com/download/symbols
         test_symbol.exe
         5FDA21D230000
SYMSRV:  UNC: c:\winsymbols\test_symbol.exe\5FDA21D230000\test_symbol.exe - path not found
SYMSRV:  UNC: c:\winsymbols\test_symbol.exe\5FDA21D230000\test_symbol.ex_ - path not found
SYMSRV:  UNC: c:\winsymbols\test_symbol.exe\5FDA21D230000\file.ptr - path not found
SYMSRV:  HTTPGET: /download/symbols/test_symbol.exe/5FDA21D230000/test_symbol.exe
SYMSRV:  HttpQueryInfo: 80190194 - HTTP_STATUS_NOT_FOUND
SYMSRV:  HTTPGET: /download/symbols/test_symbol.exe/5FDA21D230000/test_symbol.ex_
SYMSRV:  HttpQueryInfo: 80190194 - HTTP_STATUS_NOT_FOUND
SYMSRV:  HTTPGET: /download/symbols/test_symbol.exe/5FDA21D230000/file.ptr
SYMSRV:  HttpQueryInfo: 80190194 - HTTP_STATUS_NOT_FOUND
SYMSRV:  RESULT: 0x80190194
SYMSRV:  BYINDEX: 0xB

```


ChkMatch.exe 是一个 Debug Info 分析工具 [http://www.debuginfo.com/tools/chkmatch.html](http://www.debuginfo.com/tools/chkmatch.html)


通过 ChkMatch.exe 工具可以得到新编译的 exe 和 pdb 的 Timestamp 和 Signature，


注意如果 Signature: {8b4b1010-1862-4691-b6f1-7d012432902e}， 对应的二进制数据顺序为：


```text
10 10 4b 8b 62 18 91 46 b6 f1 7d 01 24 32 90 2e

```


用 Hex Editor 对新的 exe 和 pdb 中的 Timestamp 的 Signature 值进行替换，再通过 Synstore 工具进行备份。

