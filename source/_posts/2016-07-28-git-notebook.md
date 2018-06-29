---
layout: post
title: "Git Notebook"
date: 2016-07-28 21:35:27 +0800
comments: true
categories: 
keywords: git-notebook, git备忘录
description: git-notebook, git备忘录
---

## 更新 `.gitignore` 文件

1. [github 官方给出的各语言 .gitignore 示例文件](https://github.com/github/gitignore)
2. 当在 git 仓库中更新了 .gitignore 文件，需要执行下面命令，才能清除那些被忽略的文件

```
git rm -r --cached .
git add .
git commit -m "update .gitignore"
```

## 修改最后一次提交

1. 有时候提交完发现漏掉了几个文件没有加，或者提交信息写错了，想要撤销刚才的提交操作，可以使用 `git commit --amend` 选项重新提交
2. 如果上一次的提交已经 `push` 到远程仓库，推到远程需要加 `-f` 参数来覆盖远程仓库，不过一般不建议这么做

## 显示提交记录的参与者列表

```
git shortlog -sn
```

显示提交记录的参与者列表，和 github 的参与者列表相同

## 标签 tag 操作

1. `git tag` 显示本地 tag 列表
2. `git tag -d [tagname]` 删除本地名为 [tagname] 的 tag
3. `git push origin :refs/tags/[tagname]` 删除远程名为 [tagname] 的 tag
4. `git tag -a [tagname] -m "xxx"`，用于创建一个含附注类型的标签，用 -a 指定标签名字，-m 选项则指定了对于的标签说明
5. `git show [tagname]`，此命令用于查看相应标签的版本信息，并连同显示打标签的提交对象
6. `git push origin [tagname]`，通过显式命令将标签推到远端仓库，一次推送所有（本地新增的）标签到远端仓库，使用 `git push origin --tags`

## 查看远程仓库详细信息

```
git remote show origin
```

## git stash 删除恢复

首先在终端输入👇命令查找所有 unreachable 的记录

```
git fsck --unreachable
```

这时候会在终端看到类似👇的输出

```
Checking object directories: 100% (256/256), done.
Checking objects: 100% (40741/40741), done.
unreachable commit 6c00def1f6db3a9453362fefd9e5d5e1edd94b8a
unreachable blob a10014bb776e27387f304cb4fb0cd1fbf26010a5
unreachable blob fa00b66289157355fd27765f5220db41e6fef2f8
unreachable commit 790134c192c7d114086cd9fe5bd8c0c395af6767
unreachable blob 03036483ed3d232a0e347b541a7759304bd3e73b
unreachable blob 5503ea6ffa56f475ee2cb5857830d2464f4c255b
unreachable blob 59039c5f01ca5580bff4a9c4ed2601004e5fac83
unreachable blob 9f03bcf7d4b33b57be6842aea99ab9878dc91c46
...
```

可以使用 `git show xxx` 命令查看具体某条记录对应的改动，找到需要的记录之后，可以直接使用 `git merge xxx` 命令，本地仓库就恢复了该条记录对应的内容。(其中 xxx 代表上面列出的记录中后面跟的 Commit Hash 值)

有可能会出现很多 unreachable 的记录，这时一个一个拿 `git show` 命名去找是否是我们需要的，会太浪费时间，这时可以利用👇命令，输出所有 unreachable 对应的提交改动

```shell
git fsck --unreachable 2&>/dev/null | while read i; do; git show `echo $i | cut -d ' ' -f 3` | head -n 6; done
```

然后在终端中搜索某些关键字，比如具体改动的代码、时间，就可以迅速定位到对应的 Hash 值，接着恢复就 OK 了。

这种方法，不单单只对误删除了 stash 记录有作用，git 一切错误操作导致删除都可以利用此方法恢复。

## git submodule 使用

[git submodule 使用小结](http://blog.gezhiqiang.com/2017/03/08/git-submodule/)




