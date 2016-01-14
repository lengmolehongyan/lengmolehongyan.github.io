---
layout: post
title: "计算代码执行时间差"
date: 2015-12-05 23:53:21 +0800
comments: true
categories: iOS开发小细节
keywords: 计算代码执行时间差
description: 计算代码执行时间差
---

开发中，有时候我们需要知道某个效果到底耗时多长时间，可能想到用 `Foundation` 框架中的 `NSDate` 类来计算，比如：

```objc
NSDate *previousDate = [NSDate date];
...
NSTimeInterval timeInterval = [[NSDate date] timeIntervalSinceDate:previousDate];
```

在 `CoreFoundation` 框架中，有一个函数返回当前系统的绝对时间，也可以用做计算时间差。如下：

```objc
CFAbsoluteTime previousTime = CFAbsoluteTimeGetCurrent();
...
CFAbsoluteTime currentTime = CFAbsoluteTimeGetCurrent();
CFAbsoluteTime timeInterval = currentTime - previousTime;
```


