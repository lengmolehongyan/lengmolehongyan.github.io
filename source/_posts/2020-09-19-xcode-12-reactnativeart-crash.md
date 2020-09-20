---
layout: post
title: "Xcode 12 ReactNativeART Crash"
date: 2020-09-19 09:59:34 +0800
comments: true
categories: 
keywords: "Xcode, ReactNative, ART, Crash"
description: "Xcode 12 ReactNativeART Crash"
---

> 本文章为排查一次崩溃的简单记录

### 0x01

项目中需要用到阴影，依赖了 [ReactNativeART](https://github.com/react-native-community/art) 库。Xcode 12 正式版发布后，立马更新了，编译 NO Error，运行 NO Error，结果点到使用阴影的地方崩了。。。

![crash](https://i.loli.net/2020/09/19/Z6LqHauJlwhcQyB.jpg)

### 0x02

`EXC_BAD_ACCESS` iOS 很常见的野指针错误，打开僵尸对象后，重新运行，看看究竟是哪个对象引起的

![zombie object](https://i.loli.net/2020/09/19/Rqyg6XQLCv4PTNp.jpg)

![crash detail](https://i.loli.net/2020/09/19/ZU1ivf7LVmhrXo2.jpg)

控制台很明确的告知，对已经释放的 color 对象调用方法，回头看崩溃所在代码 `_shadow = shadow;`，查看头文件 `ARTShadow` 为结构体，内部引用了 `UIColor` 对象

```objc
typedef struct {
  CGSize offset;
  CGFloat blur;
  UIColor* color;
} ARTShadow;
```

结构体一般情况下是存储在栈中的（手动分配内存构造存储在堆中），对于存储在栈区的变量，编译器会自动做好垃圾回收。借助 Xcode 编译器特性，会为包含 OC 对象的结构体自动生成隐式的构造、析构函数，即结构体实例创建时，结构体中的所有成员清零，结构体实例销毁时，会自动将其持有的 OC 对象的指针置为 nil 以减少该对象的引用计数

因此，怀疑 `ARTNode` 类的成员变量 `@property (nonatomic, assign) ARTShadow shadow;` ，该结构体持有的 OC 对象存在过早释放问题，触发野指针错误。事情是这样吗？写个 [demo](https://github.com/lengmolehongyan/StructCrashVerify) 简单测试下

### 0x03

![verify](https://i.loli.net/2020/09/20/EU8F2qHBVx61AXM.png)

如上图，只要多次访问持有 OC 对象的结构体， Xcode 12.0 版本百试百崩，应该是 Xcode 新版本的问题了

如何解决呢，改造该结构体成员，让其所有成员只为基本数据类型就不会崩溃了

```objc
typedef struct {
  CGSize offset;
  CGFloat blur;
  CGFloat alpha;
  CGFloat color;
} ARTShadow;
```

针对该阴影 RN 库的解决办法，我已经给官方提了 [PR](https://github.com/react-native-community/art/pull/70)，读者可以去 PR 查看详细代码

### 0x04

经过这一茬，Xcode 告诫我们尽量不要在结构体中持有 OC 对象成员，指不定哪天蹦出个莫名其妙的 Crash...