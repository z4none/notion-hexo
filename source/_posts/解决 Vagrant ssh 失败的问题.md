---
updated: '2018-07-09 08:00:00'
categories: code
excerpt: "虚拟机启动后运行\_vagrant ssh\n\_出错, 相关错误信息如下:Permission denied (publickey)."
date: '2018-07-09 08:00:00'
tags:
  - Dev
urlname: vagrant-ssh-failure
title: 解决 Vagrant ssh 失败的问题
---

# 环境

- Windows 10
- Vagrant 2.1.1
- OpenSSH_for_Windows_7.6p1, LibreSSL 2.6.4

# 现象


虚拟机启动后运行 `vagrant ssh` 出错, 相关错误信息如下 `vagrant ssh - --v`:


```text
debug1: Trying private key: D:/Docker/.vagrant/machines/default/virtualbox/private_key
debug3: Bad permissions. Try removing permissions for user: S-1-5-11 on file D:/Docker/.vagrant/machines/default/virtualbox/private_key.
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions for 'D:/Docker/.vagrant/machines/default/virtualbox/private_key' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "D:/Docker/.vagrant/machines/default/virtualbox/private_key": bad permissions
debug2: we did not send a packet, disable method
debug1: No more authentication methods to try.
vagrant@127.0.0.1: Permission denied (publickey).

```


issue: [https://github.com/hashicorp/vagrant/issues/9433](https://github.com/hashicorp/vagrant/issues/9433)


# 问题分析


openssh 客户端连接虚拟机时对 private_key 文件权限进行检查发现它可被其他用户访问


# 解决方法


在 private_key 文件的属性 / 安全中设置所有者为当前用户并取消所有继承, 当前用户完全控制

