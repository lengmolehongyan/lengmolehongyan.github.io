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

<!--more-->

![jenkins.war]({{root_url}}/images/QQ20160113-0@2x.png)

下载完成后，终端进入到 jenkins.war 所在文件夹，执行以下命令：

```sh
$ java -jar jenkins.war --httpPort=8888
```

可以创建一个 bash/zsh alias 来给上述指令起一个别名，示例如下：

```bash
alias jenkins="java -jar 此处为jenkins.war文件所在路径"
```

如果使用的是 bash，在用户目录下的 `.bashrc` 文件中添加上面这句，别名就起好了，如果使用的是 zsh，则在用户目录下的 `.zshrc` 文件增加。之后，在终端，直接输入 `jenkins` 指令就可以启动 jenkins 。

如果出现以下提示，说明需要升级 Java 版本，Jenkins 需要至少 Java7 及以后的版本，可以在此页面下载 [Java](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html) 。

![错误提示]({{root_url}}/images/2016011315-19-43@2x.png)

上面在终端输入的命令中，`httpPort` 指定的是 Jenkins 所使用的本机 http 端口号，可以自行修改，等待 Jenkins 完全启动后，终端会有如下提示：

```sh
...
信息: Jenkins is fully up and running
```

现在在浏览器打开 [http://localhost:8888/](http://localhost:8888/)，就可以看到本机 Jenkins 的界面了。

## Jenkins 配置

Jenkins 默认没有安装 `git` 插件，需要手动安装。如下图，在 Jenkins 的界面左侧，依次点击系统管理，管理插件，在可选插件下，筛选 `git`，然后勾选 Git plugin，Git server plugin 和 Git client plugin，点击下载待重启后安装按钮，等待插件下载安装成功后，重启 Jenkins 就可以了。

![管理插件]({{root_url}}/images/QQ20160113-2@2x.png)

![安装 git 插件]({{root_url}}/images/QQ20160113-1@2x.png)

安装过程中如果遇到下面这种错误，提示插件下载地址错误导致安装失败，从错误信息中拷贝重定向后的地址手动下载，下载完成后进入插件管理，选择高级，然后上传插件安装就可以了。

![安装插件失败]({{root_url}}/images/QQ20160113-3@2x.png)

![手动安装插件]({{root_url}}/images/QQ20160114-0@2x.png)

在 Jenkins 系统管理，系统设置中可以配置系统管理员邮件地址和邮件通知，Jenkins 可以在适当的时机发送邮件通知，发送邮件使用的是 `SMTP` 协议。在设置邮箱时，Jenkins 管理员邮箱要与 `SMTP` 中设置的发送邮箱为同一个邮箱。这里配置完成后，可以发送测试邮件，测试是否配置成功。

![配置系统管理员邮件地址]({{root_url}}/images/QQ20160114-1@2x.png)

![配置邮件通知]({{root_url}}/images/QQ20160114-2@2x.png)

## 新建项目

Jenkins 默认使用当前用户 `.ssh` 目录下的公私钥来进行 `git` 的相关操作。在 Jenkins 首页，点击新建，输入项目名称，选择构建项目的属性，然后点击 OK，进入项目配置页面。

![创建项目]({{root_url}}/images/QQ20160114-3@2x.png)

在项目配置页面，找到源码管理，配置项目的远程仓库，填入项目的远程仓库 `git` 地址，以及编译构建项目的分支。

![项目配置]({{root_url}}/images/QQ20160114-4@2x.png)

下一步就是对项目编译的设置，在项目配置最下方的构建选项，点击增加构建步骤，可以选择通过 `shell` 脚本编译，也可以使用 Jenkins 自带的 Xcode 插件（需要安装 Xcode 插件）。编写脚本，可以直接使用 Xcode 的 xcodebuild 来写，也可以直接使用 Facebook 的 [xctool](https://github.com/facebook/xctool) 。

![项目编译设置]({{root_url}}/images/QQ20160114-5@2x.png)

项目成功编译以后，可以设置编译构建出来的 ipa 文件保存位置，同时可以设置当编译构建失败时的邮件提醒。

![编译后操作设置]({{root_url}}/images/QQ20160114-6@2x.png)

上述所有操作完成之后，点击应用并保存，回到测试项目首页，便可以编译构建项目了，项目如果需要修改配置，可以直接在配置里面修改。

![项目创建完成]({{root_url}}/images/QQ20160114-7@2x.png)

## TODO

 接下来就是学习 Facebook 的 xctool 的使用......








