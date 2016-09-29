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




