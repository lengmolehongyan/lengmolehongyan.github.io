---
layout: post
title: "CodePush 打包 && 发布 && 调整灰度比例相关脚本实现"
date: 2019-09-06 14:40:36 +0800
comments: true
categories: 
keywords: codepush，打包
description: CodePush 打包 && 发布 && 调整灰度比例相关脚本实现
---

## React Native Bundle 打包命令

### iOS

```shell
$ node node_modules/react-native/local-cli/cli.js bundle --assets-dest ./bundles/ios/ --bundle-output ./bundles/ios/main.jsbundle --dev false --entry-file index.js --platform ios
```

### Android

Android bundle 包命名必须为 index.android.bundle

```shell
$ node node_modules/react-native/local-cli/cli.js bundle --assets-dest ./bundles/android/ --bundle-output ./bundles/android/index.android.bundle --dev false --entry-file index.js --platform android
```

上述命令中的 `node node_modules/react-native/local-cli/cli.js bundle` 与 `react-native bundle` 相同作用，前提需在一个 react-native 工程中。

## Bundle 及资源文件压缩

如上 react native bundle 打包后，Android 平台会生成如下文件及目录

```shell
.
└── android
    ├── drawable-hdpi
    ├── drawable-mdpi
    ├── drawable-xhdpi
    ├── drawable-xxhdpi
    ├── drawable-xxxhdpi
    ├── index.android.bundle
    └── index.android.bundle.meta

6 directories, 2 files
```

iOS 平台会生成如下文件及目录

```shell
.
└── ios
    ├── assets
    ├── main.jsbundle
    └── main.jsbundle.meta

2 directories, 2 files

```

<!--more-->

### 压缩

自建 CodePushServer 基础平台–热部署发布平台–创建应用发布，上传代码包时需要是 zip 压缩文件，如下命令可将 ios/android 目录下递归所有文件并压缩

```shell
# fileName 命名规则：当前 Build 号 && 当前平台 && 当前打包分支.zip
$ zip -r ./fileName ./ios/*
$ zip -r ./fileName ./android/*
```

打包压缩后，需要供内网下载，在 MacPro 上开启 Mac 自带 Apache 服务器，有两种简易方式：

1. Mac 开启自带 Apache 服务器（[https://www.jianshu.com/p/b1731c4a00ec](https://www.jianshu.com/p/b1731c4a00ec)）
    * 缺点：手动配置
2. VirtualHostX（[https://clickontyler.com/virtualhostx/](https://clickontyler.com/virtualhostx/)）
    * 优势：安装完 App 只用无脑点便可以开启

这里采用的第 2 种方式，配置完后，将刚才压缩后的文件复制到 Apache 服务器相应目录，内网访问相应地址便可以下载

[http://macpro.10.11.9.205.xip.io:8080/CodePush/](http://macpro.10.11.9.205.xip.io:8080/CodePush/)

```shell
# /opt/Sites/ 为 Apache 服务器根目录
$ cp -rf ./bundles/fileName /opt/Sites/CodePush/
```

## CodePush

### CodePush 发布命令

```shell
# appName 为 CodePush 所注册应用名称
# platform 为本次对哪个平台发布 ios/android
# nativeVersion 为本次对该平台哪个原生版本发布
$ code-push release-react appName platform -t nativeVersion --dev false -d Production -m --des "xxxxxx"
```

### 获取 iOS/Android 平台最新 CodePush Label

1. 获取某个应用 CodePush 发布记录
2. 查找匹配 Active 所有行
3. 在所有行的结果中取出最后一行
4. 取出最后一行第二个变量

综合命令如下

```shell
# appName 为 CodePush 所注册应用名称
$ code-push deployment history appName Production | grep 'Active' | tail -1 | awk '{print $2}'
```

### CodePush 更新灰度比例

依照上一条，可以得出最新发布的 CodePush Label，如果该次发布为灰度发布，想增大灰度比例时，参照如下命令

```shell
# appName 为 CodePush 所注册应用名称
# vXXX 为调整 CodePush 灰度比例对应的 CodePush Label
# -r 后面跟的数字为 0~100 之间
$ code-push patch appName Production -l vXXX -r 100
```

### JSON 输出

查看 CodePush 发布记录可以在命令后追加 `format` 格式化输出

```shell
# appName 为 CodePush 所注册应用名称
$ code-push deployment history appName Production --format=json
```

### JSON 解析

jq 是一个可以在终端 shell 中解析 json 输出的工具

官方文档：[https://stedolan.github.io/jq/](https://stedolan.github.io/jq/)

```shell
$ brew install jq
```

相关解析教程，可以参照官方教程：[https://stedolan.github.io/jq/tutorial/](https://stedolan.github.io/jq/tutorial/)

结合 CodePush 发布记录的 json 输出和 jq 工具，可以快速得出 CodePush 发布最新版本的更新安装人数

```shell
# appName 为 CodePush 所注册应用名称
# 获取当前发布记录总个数
$ count=$(code-push deployment history appName Production --format=json | jq 'length')
# 获取最新一次发布详细数据
$ code-push deployment history appName Production --format=json | jq ".[$count-1]"
# 获取最新一次发布安装人数
$ code-push deployment history appName Production --format=json | jq ".[$count-1]" | jq '.metrics.downloaded'
```

## 获取需求对应开发人员

当前客户端需求上线前，需要接受 Merge Request 合并代码至线上分支，提 MR 的人就是需求对应开发人员

### 获取 Merge Request ID

在 gitlab 创建 MR 时，会基于该工程生成本次 merge request id，当该 MR 被接受 merge 后，自动生成一次提交记录，而提交记录的详细信息中就有对应的 merge request id

自动生成的提交记录类似如下

```shell
Merge: 44dd8c51 61632fed
Author: xx 
Date:   Thu Sep 5 15:44:25 2019 +0800

    Merge branch 'xxxx' into 'online'

    xxxxxx merge request title
    See merge request !72
```

依据样例，便可以得出获取 MR ID 的方法：

1. 基于线上代码分支，获取最后一次提交记录
2. 查找 See merge request
3. 解析得出 merge request id

综合命令如下

```shell
# 多一次 `sed 's/ //g'` 原因是解析出来的结果包含多余空格
$ git log -1 Head | grep 'See merge request' | sed 's/See merge request !//g' | sed 's/ //g'
```

### GitLab API

拿到 merge request id 后，通过 gitlab api 可以获取该 MR 的详细信息，从中便可以获取到本次提 MR 的对应开发人员

gitlab api 官方文档：[https://docs.gitlab.com/ee/api/](https://docs.gitlab.com/ee/api/)

自建 gitlab 则可以通过 gitlab 域名后追加 help/api 访问 [https://xxx.xxx.xxx/help/api/README.md](https://xxx.xxx.xxx/help/api/README.md)

Merge Request 相关 API 文档：[https://docs.gitlab.com/ee/api/merge_requests.html](https://docs.gitlab.com/ee/api/merge_requests.html)

API 访问鉴权方式文档：[https://docs.gitlab.com/ee/api/#authentication](https://docs.gitlab.com/ee/api/#authentication)

获取提 MR 的对应开发人员方法：

1. 获取 merge request id
2. 通过 gitlab 接口请求本次 merge request 详细信息
    * 请求接口地址中的 project id 从 gitlab 仓库 Settings → Project settings → Project ID 获取
3. jq 解析第 2 步的 json 输出，author 就是对应开发人员信息

命令如下：

```shell
$ mergeRequestId=`git log -1 Head | grep 'See merge request' | sed 's/See merge request !//g' | sed 's/ //g'`
# project id 从 gitlab 仓库 Settings → Project settings → Project ID 获取
$ url="https://xxx.xxx.com/api/v4/projects/projectid/merge_requests/"${mergeRequestId}
# PRIVATE-TOKEN 获取方式参照上述文档
$ curl -H 'PRIVATE-TOKEN: xxxxxxxxx' -H 'content-type: application/json' -H 'Accept: */*' -H 'Cache-Control: no-cache' --compressed ${url} | jq '.author.name' | sed 's/\"//g'
```

## 钉钉机器人相关脚本

钉钉机器人相关脚本可参照钉钉官方文档：[https://ding-doc.dingtalk.com/doc#/serverapi2/qf2nxq](https://ding-doc.dingtalk.com/doc#/serverapi2/qf2nxq)
