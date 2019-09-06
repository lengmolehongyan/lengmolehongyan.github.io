---
layout: post
title: "iOS 开发常用目录备忘"
date: 2018-01-08 12:00:48 +0800
comments: true
categories: ios开发小细节
keywords: #keyword
description: #seo 的 desc
---

### DerivedData

`/Users/xxxx/Library/Developer/Xcode/DerivedData`

编译项目所产生的缓存目录

### DeviceSupport

`/Users/xxxx/Library/Developer/Xcode/iOS DeviceSupport`

当前运行的 Xcode 版本所支持真机调试的系统版本（例：当 Xcode 未升级想调试系统版本高的）

低版本 Xcode 调试高版本 iOS 系统时，只需从进入高版本 Xcode 下面目录拷出对应高版本系统镜像，然后复制到低版本 Xcode 对应目录，完成后重启 Xcode，等读条完，便可以调试了

`/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport/`

### Archives

`/Users/xxxx/Library/Developer/Xcode/Archives`

Xcode Archive 打包所在目录

### CocoaPods

`/Users/xxxx/.cocoapods/repos/master`

CocoaPods git 主仓库所在目录

### 模拟器

`/Library/Developer/CoreSimulator/Profiles/Runtimes`

Xcode 旧版本模拟器所在目录


