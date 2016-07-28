---
layout: post
title: "directory not found for option"
date: 2015-09-23 10:47:48 +0800
comments: true
categories: 
keywords: iOS9, 适配, 教程
description: ios9适配
---

在 Xcode7 下编译项目，报了如下警告：

```
ld: warning: directory not found for option 
'-F/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS9.0.sdk/Developer/Library/Frameworks'
```

这是因为 Xcode7 将 `Frameworks` 位置改变了。

<!--more-->

点击项目，选择 TARGETS 中的工程名 Tests，然后选择 Build Settings，在 Search Paths 下，可以看到此时 `Framework Search Paths` 路径，此路径为错误路径。

![](http://i3.tietuku.com/40d46fe638434b1e.png)

双击路径，删除 `$(SDKROOT)/Developer/Library/Frameworks`，增加 `$(PLATFORM_DIR)/Developer/Library/Frameworks`，此时路径就已经修改成功。

![](http://i3.tietuku.com/6b824df6fd5bc002.png)

修改完成之后，重新编译项目，此警告就修复了。


