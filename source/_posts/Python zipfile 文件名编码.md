---
updated: '2019-11-12 08:00:00'
categories: code
excerpt: 问题： 使用 Python3 的内置模块 zipfile 解压 zip 文件时，中文文件名出现乱码
date: '2019-11-12 08:00:00'
tags:
  - Web
urlname: python-zipfile-filename
title: Python zipfile 文件名编码
---

问题： 使用 Python3 的内置模块 zipfile 解压 zip 文件时，中文文件名出现乱码


# 原因


zip 文件没有标准的文件名编码定义，有些 zip 会采用 UTF-8 编码，而有些工具（Winzip）统一使用 CP437 对文件名进行编码（DOS Latin），而不是当前操作系统的文件名编码，UTF-8 和 CP437 的区别是看是否设置了文件头标志位 EFS：

- 如果为 0 表示采用 CP437
- 如果为 1 表示采用 UTF-8

python 3.6 的 zipfile 解压时的处理如下：


```text
def _decodeFilename(self):
    if self.flag_bits & 0x800:
        return self.filename.decode('utf-8')
    else:
        return self.filename

```


如果有 UTF-8 标志，则解码成字符串，否则保持为 bytes 由用户处理


# 解决方案


碰到此类文件，可以将文件名先编码为 CP437， 然后再解码为 CP936 即可获得正确结果


```text
unzipped = zipfile.ZipFile(zip_file_path)
for filename in unzipped.namelist():
    if isinstance(item, bytes):
        filename = filename.encode('cp437').decode('cp936')

```


# 参考


[ZIP File Format Specification](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT)


[https://stackoverflow.com/questions/37723505/namelist-from-zipfile-returns-strings-with-an-invalid](https://stackoverflow.com/questions/37723505/namelist-from-zipfile-returns-strings-with-an-invalid-encoding)

