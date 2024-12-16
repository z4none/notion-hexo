---
updated: '2018-01-15 00:00:00'
categories: code
excerpt: eXosip 是 osip 的一个封装，代码简洁易于使用，支持的平台广泛，但编译起来难以一次成功，下面记录一下我今天在 VC2017 环境下的编译过程
date: '2018-01-15 00:00:00'
tags:
  - C++
urlname: vc2018-exosip
title: 在 VC 2018 下编译 eXosip
---

eXosip 是 osip 的一个封装，代码简洁易于使用，支持的平台广泛，但编译起来难以一次成功，下面记录一下我今天在 VC2017 环境下的编译过程


从 [http://www.antisip.com/download/exosip2/](http://www.antisip.com/download/exosip2/) 下载 osip 和 exosip 本文用的版本是 5.0，分别为 libexosip2-5.0.0.tar.gz 和 libosip2-5.0.0.tar.gz


还需从 [http://c-ares.haxx.se/](https://c-ares.haxx.se/) 下载 c-ares-1.13.0.tar.gz


将下载的压缩包解压到各自文件夹，并重命名为： * c-ares * exosip * osip


其中 exosip 中缺少一个 eXrefer_api.c 文件，从 [https://github.com/aurelihein/exosip/blob/master/src/eXrefer_api.c](https://github.com/aurelihein/exosip/blob/master/src/eXrefer_api.c) 下载到对应路径中


打开 \exosip\platform\vsnet 中的 eXosip.sln，可以加载 4 个工程，这里需要选择 Windows SDK 版本 8.1（如果选择升级到 Windows SDK 10 编译时会出现 winnt.h 的 LINE 18017 错误，貌似是 VS 的 BUG？）


工程成功加载后直接编译会出现很多错误，可按以下方法解决：


## osipparser2

- snprintf 重定义，注释 internal.h 的 58 行
- WINAPI_FAMILY_ONE_PARTITION宏定义错误，在 61 行之后添加：

	```text
	#define WINAPI_FAMILY_ONE_PARTITION(PartitionSet, Partition) ((WINAPI_FAMILY & PartitionSet) == Partition)
	
	```


## osip2

- 同 osipparser2
- timespec 重定义，预处理宏添加 HAVE_STRUCT_TIMESPEC

## c-ares


编译没错误，但是需要在项目中添加文件： - ares_create_query.c - ares_platform.h - ares_platform.c


否则在使用时会报链接错误： error LNK2019: ares_getplatform，该符号在函数_get_DNS_Registry中被引用


## eXosip

- snprintf 重定义，注释 eXosip2.h 的 73 行
- OPENSSL 相关错误，安装 OPENSSL 或者去除预处理宏 HAVE_OPENSSL_SSL_H、TSC_SUPPORT

---


编译成功后在项目中使用，需要增加以下 lib


```text
#pragma comment(lib, "Ws2_32.lib")
#pragma comment(lib, "Qwave.lib")
#pragma comment(lib, "delayimp.lib")
#pragma comment(lib, "Dnsapi.lib")
#pragma comment(lib, "eXosip.lib")
#pragma comment(lib, "libcares.lib")
#pragma comment(lib, "osip2.lib")
#pragma comment(lib, "osipparser2.lib")
```

