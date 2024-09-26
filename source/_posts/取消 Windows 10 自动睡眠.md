---
updated: '2016-05-02 08:00:00'
categories: code
excerpt: 最近 Windows 10 更新后，出现了一个令人困扰的问题，即使在高性能模式，它也会在 2 分钟无操作时自动睡眠
date: '2016-05-02 08:00:00'
tags:
  - Windows
urlname: cancel-win10-sleep
title: 取消 Windows 10 自动睡眠
---

最近 Windows 10 更新后，出现了一个令人困扰的问题，即使在高性能模式，它也会在 2 分钟无操作时自动睡眠


经过 Google 解决方法如下：

1. 用管理员权限修改注册表：将

```text
HKEY_LOCAL_MACHINE\
    SYSTEM\
        CurrentControlSet\
            Control\
                Power\
                    PowerSettings\
                        238C9FA8-0AAD-41ED-83F4-97BE242C8F20\
                            7bc4a2f9-d8fc-4469-b07b-33eb785aaca0
```


下的 ‘Attributes’ 改为数字 2

1. 打开 电源选项 / 更改计划设置（当前使用的计划） / 更改高级电源设置

点击 ‘更改当前不可用的设置’ 将 ‘睡眠’ / ‘无人参与系统睡眠超时’ 中的时间修改到合适，比如 60 分钟


经过以上修改，2 分钟自动睡眠的坑爹特性终于取消掉了

