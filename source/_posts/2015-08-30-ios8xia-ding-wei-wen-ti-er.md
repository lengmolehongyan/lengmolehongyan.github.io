---
layout: post
title: "iOS8 以上定位问题<二>"
date: 2015-08-30 00:19:06 +0800
comments: true
categories: 
---

接上篇，我们已经可以在系统版本 iOS8 以上完成定位功能，但是会有一个潜在 bug 存在。

进入设置应用，往下滑动找到自己的 app，进入，点击 **位置** 单元格，设置应用会彻底奔溃，回到桌面了。

<!--more-->

![点击位置崩溃](http://i3.tietuku.com/8d9493bef4f6a786.png)

经过一番 Google 和 Stack Overflow 后，问题就出现在 `info.plist` 文件中的两个缺省字段。

在上一篇博客中，我将 `NSLocationWhenInUseUsageDescription` 和 `NSLocationAlwaysUsageDescription` 的类型设置为 `Boolean`，值设置为 `YES`，而正确做法是类型设置为 `String`，值填写一些描述信息，比如：请点击允许以允许访问，若不允许将无法使用 XX 功能。这里面所填写的文字，也会自动出现在位置页面的应用程序说明之后。

![应用程序说明](http://i3.tietuku.com/3881905e46802e76.png)

完成之后，重新编译运行，再去设置那，怎么点都不会崩溃了！Over！









