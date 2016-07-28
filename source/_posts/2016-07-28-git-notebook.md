---
layout: post
title: "git-notebook"
date: 2016-07-28 21:35:27 +0800
comments: true
categories: 
keywords: git-notebook, git备忘录
description: git-notebook, git备忘录
---

###### 更新 `.gitignore` 文件

1. [github 官方给出的各语言 .gitignore 示例文件](https://github.com/github/gitignore)
2. 当在 git 仓库中更新了 .gitignore 文件，需要执行下面命令，才能清除那些被忽略的文件

```
$ git rm -r --cached .
$ git add .
$ git commit -m "update .gitignore"
```

###### 修改最后一次提交的注释信息

1. `git commit --amend` 可以修改最后一次提交的注释信息
2. 如果上一次的提交已经 `push` 到远程仓库，推到远程需要加 `-f` 参数来覆盖远程仓库，不过一般不建议这么搞

###### 显示提交记录的参与者列表

`git shortlog -sn`

显示提交记录的参与者列表，和 github 的参与者列表相同

