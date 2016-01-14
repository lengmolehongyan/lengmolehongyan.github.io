---
layout: post
title: "解决 Xcode 升级插件失效"
date: 2015-12-03 23:50:08 +0800
comments: true
categories: iOS开发小细节
keywords: 解决 Xcode 升级插件失效
description: 解决 Xcode 升级插件失效
---


* 打开终端，输入以下代码获取到 `DVTPlugInCompatibilityUUID`

<p></p>

```
defaults read /Applications/Xcode.app/Contents/Info DVTPlugInCompatibilityUUID
```

* 然后输入如下命令，最后一项是上一步获取到的 `DVTPlugInCompatibilityUUID`

<p></p>

```
find ~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins -name Info.plist -maxdepth 3 | xargs -I{} defaults write {} DVTPlugInCompatibilityUUIDs -array-add 此处为上一步获得的 UUID
```

* 终端执行完上面命令后，打开 Xcode，会有下图提示，点击 `Load Bundles`，就会加载 Xcode 插件目录中已安装的插件。

<p></p>

<!--more-->

![Load Bundles]({{ root_url }}/images/QQ20151204-0@2x.png)

~~~ 😊 OVER 😊 ~~~


