---
updated: '2016-11-07 00:00:00'
categories: code
excerpt: 在此记录 Ionic2 学习过程中遇到的问题
date: '2016-11-07 00:00:00'
tags:
  - Web
urlname: ionic-study
title: Ionic 填坑
---

在此记录 Ionic2 学习过程中遇到的问题


# 不能自定义 Splash 和 Icon


新建一个项目后通过 ionic resources 命令生成了各分辨率下的 Splash 和 Icon 文件，


但是 build 成 apk 后发现并没有生效


原因是目前版本的 Ionic 2.0.0-rc2 / CLI Version: 2.1.0-beta.3 在 build 时直接将 res 文件夹生成到了项目根目录下


解决方法是将 res 文件夹 copy 到 \platforms\android\res 后重新编译


# mergeDebugResources 时出现错误


原因是 ionic resources 生成的图片没有满足 .9.png 要求


解决方法是 通过在 `platform/android/build.gradle` 中的 android {} 中添加


```text
aaptOptions {
    cruncherEnabled = false
}
```

