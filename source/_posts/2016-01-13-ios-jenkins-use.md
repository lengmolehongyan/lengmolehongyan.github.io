---
layout: post
title: "iOS 开发使用 Jenkins 搭建 CI 服务器"
date: 2016-01-13 18:57:16 +0800
comments: true
categories: 
keywords: iOS开发使用Jenkins搭建CI服务器
description: iOS开发使用Jenkins搭建CI服务器
---

## 准备

打开 [Jenkins 官网](http://jenkins-ci.org/)，官网右侧可以下载最新版本的 [jenkins.war](http://mirrors.jenkins-ci.org/war/latest/jenkins.war)。

![jenkins.war]({{root_url}}/images/QQ20160113-0@2x.png)

下载完成后，终端进入到 jenkins.war 所在文件夹，执行以下命令：

```
java -jar jenkins.war --httpPort=8888
```

如果出现以下提示，说明需要升级 Java 版本，Jenkins 需要至少 Java7 及以后的版本，可以在此页面下载 [Java](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html) 。

![错误提示]({{root_url}}/images/2016011315-19-43@2x.png)

上面在终端输入的命令中，`httpPort` 指定的是 Jenkins 所使用的本机 http 端口号，可以自行修改，等待 Jenkins 完全启动后，终端会有如下提示：

```
...
信息: Jenkins is fully up and running
```

现在在浏览器打开 [http://localhost:8888/](http://localhost:8888/)，就可以看到本机 Jenkins 的界面了。

## Jenkins 配置

Jenkins 默认没有安装 git 插件，需要手动安装。如下图，在 Jenkins 的界面左侧，依次点击系统管理，管理插件，在可选插件下，筛选 git，然后勾选 Git server plugin 和 Git client plugin，点击下载待重启后安装按钮，等待插件下载安装成功后，重启 Jenkins 就可以了。

![管理插件]({{root_url}}/images/QQ20160113-2@2x.png)

![安装 git 插件]({{root_url}}/images/QQ20160113-1@2x.png)



