---
updated: '2014-08-08 00:00:00'
categories: code
excerpt: 打开 AcFun 首页自动签到，Chrome + TamperMonkey 测试通过
date: '2014-08-08 00:00:00'
tags:
  - JavaScript
urlname: chrome-acfun
title: 打开 AcFun 首页自动签到
---

打开 AcFun 首页自动签到，Chrome + TamperMonkey 测试通过


```text
// ==UserScript==
// @name       ACFun 每日签到
// @namespace  http://z4none.me
// @version    0.1
// @description  打开 Acfun 时自动签到
// @match      http://www.acfun.tv/
// @copyright  2014, z4none@gmail.com
// ==/UserScript==

$(function() {
    var date = new Date();
    var today = date.toDateString();
    var ac = $.cookie('auto-checkin');
    var box = $("#area-search-guide [name='query']");

    console.log("上次签到 : ", ac);
    if(ac && ac == today) return;

    box.val("正在签到 ...");
    console.log("正在签到 ...");

    $.post("/member/checkin.aspx").done(function(data){
        if(data.success){
            box.val(data.result);
            console.log(data.result);
        }
        else{
            box.val(data.result);
            console.log(data.result);
        }
        $.cookie('auto-checkin', today);
    });
});
```

