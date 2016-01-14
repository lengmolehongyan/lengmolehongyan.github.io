---
layout: post
title: "Xcode 项目忽略警告"
date: 2015-12-09 23:26:54 +0800
comments: true
categories: iOS开发小细节
keywords: Xcode 忽略警告
description: Xcode 忽略警告
---

对于一个有强迫症的我，每次 `⌘B` Build 项目时，发现一个 <img src="{{root_url}}/images/DVTStatus-Warning@2x.png" width="30"> 警告都要点进去修复了，然而，对于一些无关紧要的警告，我们是否可以选择忽略这个警告，让 Xcode 不提示呢？答案当然是可以的。

下面，就介绍一下在项目中忽略警告的三个地方：

<!--more-->

## 在源文件中忽略警告

在一些第三方库中，总能看到下面这段代码的身影，这就是用于忽略某个警告

```objc
#pragma clang diagnostic push
#pragma clang diagnostic ignored "警告标识符"
...
...
#pragma clang diagnostic pop
```

用法很简单，比如在控制器的 `-viewDidLoad` 中写了句创建一个 `eTestView` 的代码

```objc
UIView *eTestView = [[UIView alloc] init];
```

Xcode 会立即报一个警告，提示我们没有使用这个变量

![Unused variable 'eTestView']({{root_url}}/images/QQ20151213-0@2x.png)

只需在创建 `eTestView` 的前后加上如下几行，黄色警告就消失了😄


```objc
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wunused-variable"
UIView *eTestView = [[UIView alloc] init];
#pragma clang diagnostic pop
```

至于警告标识符，鼠标点几下就可以找到。

如下图，右击某个警告，选择 `Reveal in Log`（有时这个选项可能是置灰状态，不能选择，可以尝试编译下项目，或者退出 Xcode 重新来一次）

![Reveal in Log]({{root_url}}/images/QQ20151213-1@2x.png)

下图红色框中，中括号内部的就是警告标识符（先要点击右上角展开警告才能看到这一大堆信息）

![警告标识符]({{root_url}}/images/QQ20151213-2@2x.png)

## 在 Build Settings 中项目全局忽略警告

在项目的 `Build Settings` 中也可以设置忽略某种或多种类型的警告，不过在这设置的影响范围就是整个项目的了，要三思而后行，不然就是给自己挖坑。

还是上面的例子，在 `Build Settings` 中找到 `Custom Compiler Flags`，双击 `Other Warning Flags`（可以配置 `Debug` 和 `Release` 环境），填入 `-Wno-unused-variable`，完成后，编译项目，项目中所有的此类型警告都没有了。

![Build Settings 中项目全局忽略警告]({{root_url}}/images/QQ20151213-4@2x.png)

这里所填写的内容规则，仅仅是在第一种方法中找到的警告标识符中的 `W` 字母后面加上 `no-` 就可以了。

## CocoaPods 导入第三方库忽略警告

通过 CocoaPods 给项目导入了一些第三方库，这些库里面或多或少会有些警告，想消除这些警告，很简单，只需在 `Podfile` 中加上这一句 `inhibit_all_warnings!`，所有通过 CocoaPods 安装的第三库的警告就没有了。







