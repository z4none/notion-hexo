---
updated: '2013-08-20 08:00:00'
categories: code
excerpt: |-
  最近需要使用 SQLite 保存数据, 使用的是 wxSqlite 的 ASE256 加密版本
  我下载的是 3.7.6.1 修改版, 需要加入到自己的工程中的只有 sqlite3.h 和 sqlite3.lib, 小改一下 CppSQLite 就能对加密功能进行支持.
date: '2013-08-20 08:00:00'
tags:
  - Dev
urlname: sqliterc-in-shell
title: Windows 下 SQLite3 shell 使用.sqliterc
---

最近需要使用 SQLite 保存数据, 使用的是 wxSqlite 的 ASE256 加密版本


我下载的是 3.7.6.1 修改版, 需要加入到自己的工程中的只有 sqlite3.h 和 sqlite3.lib, 小改一下 CppSQLite 就能对加密功能进行支持. wxSQLite 还自带有对应编译好的 sqlite3shell.exe 命令行工具, 要在命令行工具中查看加密的数据库, 需要在打开数据库文件后执行


```text
pragma key="密码"

```


否则会提示


```text
Error: file is encrypted or is not a database

```


但是每次打开某个加密数据库都需要输入这个命令很麻烦, sqlite 支持用户配置文件 .sqliterc, 在此文件中可以写一些通用的配置命令.sqliterc 需要保存在用户目录(在 Windows 下是类似于 C:\User\Administrator 的路径), 但是我们的指定数据库密码的命令肯定不能这么指定为全局的. 通过查看 shell.c 可以知道, sqlite 实际上取的是 USERPROFILE 环境变量中的值, 那么我们可以在数据库同文件夹中写一个批处理:


```text
set USERPROFILE=%~dp0
sqlite3shell database.db

```


在同文件夹中创建 .sqliterc


```text
.mode column
.header on
.nullvalue NULL
pragma key="你的密码";

```


sqlite3shell 启动时显示 “– Loading resources from 某某路径/.sqliterc” 说明配置被成功载入了, 此时执行命令 “.databases” 即可查看数据库信息

