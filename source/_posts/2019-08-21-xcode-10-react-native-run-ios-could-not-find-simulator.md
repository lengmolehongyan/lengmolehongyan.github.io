---
layout: post
title: "Xcode 10 react-native run-ios could not find simulator"
date: 2019-08-21 17:03:05 +0800
comments: true
categories: 
keywords: Xcode 10, react-native, could not find simulator
description: Xcode 10 react-native run-ios could not find simulator
---

### 问题

升级 Xcode 10 后，终端编译运行 0.55.4 版本的 React Native 项目

`react-native run-ios --simulator "iPhone 8"`

然后会报错，找不到模拟器

`Could not find xxx simulator`

### 解决办法

1. 进入 `/node_modules/react-native/local-cli/runIOS/` 目录；
2. 打开 `findMatchingSimulator.js` 文件；
3. 修改 check 模拟器相关代码如下：


原代码

```JavaScript
if (!version.startsWith('iOS') && !version.startsWith('tvOS')) {
    continue;
}
```

替换为


```JavaScript
if (!version.startsWith('com.apple.CoreSimulator.SimRuntime.iOS') && !version.startsWith('com.apple.CoreSimulator.SimRuntime.tvOS')) {
    continue;
}
```

    


