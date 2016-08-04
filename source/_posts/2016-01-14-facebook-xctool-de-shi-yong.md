---
layout: post
title: "Facebook xctool 的使用"
date: 2016-01-14 18:18:43 +0800
comments: true
categories: 
keywords: facebook, xctool, 持续集成
description: 本文是 Facebook xctool 工具的使用介绍，学习并使用以替代 Apple 官方的 xcodebuild，对项目的持续集成 CI 提供便利。
---

## xctool

**xctool** 相比苹果的 **xcodebuild**，可以更加简单方便地编译构建和测试 iOS、Mac 项目，对可持续集成 CI 尤其有用。

## 特点

**xctool** 在 xcodebuild 的基础上添加了一些额外的功能：

* 格式化输出构建和测试项目结果：

	xctool 将所有构建和测试项目的结果格式化为 JSON 对象，如果你正在构建一个可持续集成系统，使用 xctool 意味着不用再正则解析 xcodebuild 的输出结果了。

	尝试一种日志输出样式，也可以自定义输出结果，或者使用 `-reporter json-stream` 命令可以查看完整的事件流。

<!--more-->

* 友好的，ANSI 标准色输出：

	xcodebuild 为每个源文件打印所有编译指令和输出，非常冗长。而 xctool 默认只在出错的地方比较详细，更容易找出问题所在。

	例：

	![](https://camo.githubusercontent.com/f4c5388651b83663ff811969c0e2099073c25484/68747470733a2f2f66706f747465725f7075626c69632e73332e616d617a6f6e6177732e636f6d2f7863746f6f6c2d7569636174616c6f672e676966)

* 更快、并行化的测试运行：

	xctool 可以视情况并行运行所有测试任务，大大加快测试工作。在 Facebook，并行运行我们已经看到 2~3 倍的加速。

	使用 `-parallelize` 选项以启用 run-tests 或者 test，更多信息请参阅 [Parallelizing Test Runs](https://github.com/facebook/xctool#parallelizing-test-runs)。

* 编写于 Objective-C：

	xctool 是使用 Objective-C 编写的，Mac OSX 和 iOS 开发者不需要学习新的语言，并且可以轻松地提交新的功能和修复 bug。我们非常欢迎提交 PR (pull request)。

## 使用必备条件

* Xcode6 或更高版本
* 需要安装 Xcode 的命令行工具（Command Line Tools），可以在 Xcode → Preferences → Downloads 安装。

## 安装

xctool 可以从 homebrew 安装

```bash
$ brew install xctool
```

也可以下载后运行 xctool.sh 命令。

## 使用

xctool 的命令和选项是 xcodebuild 的一个扩展，在大多数情况下，你只需要替换 **xcodebuild** 为 **xctool**，会如期运行，但有更具吸引力的输出。

可以通过如下命令获取帮助和完整的配置列表：

```bash
path/to/xctool.sh -help
```

### 编译

使用 xctool 编译项目和使用 xcodebuild 是一样的。

如果是使用 workspaces 和 schemes ：

```bash
path/to/xctool.sh \
	-workspace YourWorkspace.xcworkspace \
	-scheme YourScheme \
	build
```

如果使用的是 projects 和 schemes：

```bash
path/to/xctool.sh \
	-project YourProject.xcodeproj \
	-scheme YourScheme \
	build
```

常见配置比如 `-configuration`，`-sdk`，`-arch` 和 xcodebuild 是相同的。

注意：xctool 不支持使用 `-target` 来编译 targets，必须使用 schemes。

### 单元测试

xctool 的 **test** 命令，可以配置 scheme 中如何编译和运行测试。也可以限制哪些单元测试运行，或者指定运行的 SDK 版本。

使用如下命令，编译运行所有单元测试：

```
path/to/xctool.sh \
	-workspace YourWorkSpace.xcworkspace \
	-scheme YourScheme \
	test
```

在指定的 target 上编译运行单元测试，使用 `-only` 配置：

```bash
path/to/xctool.sh \
	-workspace YourWorkSpace.xcworkspace \
	-scheme YourScheme \
	test -only SomeTestTarget
```

可以进一步配置只运行特定的单元测试类：

```bash
path/to/xctool.sh \
	-workspace YourWorkSpace.xcworkspace \
	-scheme YourScheme \
	test -only SomeTestTarget:SomeTestClass
```

甚至只运行某个单元测试类的某个方法：

```bash
path/to/xctool.sh \
	-workspace YourWorkSpace.xcworkspace \
	-scheme YourScheme \
	test -only SomeTestTarget:SomeTestClass/testSomeMethod
```

也可以指定匹配前缀的类和方法：

```bash
path/to/xctool.sh \
	-workspace YourWorkSpace.xcworkspace \
	-scheme YourScheme \
	test -only SomeTestTarget:SomeTestClassPrefix*,SomeTestClass/testSomeMethodPrefix*
```

也可以在指定 SDK 版本上运行单元测试：

```bash
path/to/xctool.sh \
	-workspace YourWorkSpace.xcworkspace \
	-scheme YourScheme \
	test -test-sdk iphonesimulator5.1
```

### 编译单元测试

当编译和运行单元测试时，有时不需要运行只是编译，可以使用 `build-tests` 指令，例：

```bash
path/to/xctool.sh \
	-workspace YourWorkSpace.xcworkspace \
	-scheme YourScheme \
	build-tests
```

可以使用 `-only` 指定在某个 target 上编译：

```bash
path/to/xctool.sh \
	-workspace YourWorkSpace \
	-scheme YourScheme \
	build-tests -only SomeTestTarget
```

### 运行单元测试

如果已经使用 `build-tests` 编译，使用 `run-tests` 就可以直接运行单元测试。这样的作用是只需编译一次，便可运行在不同 SDK 版本上。

运行所有单元测试，你将使用：

```bash
path/to/xctool.sh \
	-workspace YourWorkSpace.xcworkspace \
	-scheme YourScheme \
	run-tests
```

和 `test` 指令一样，可以通过 `-only` 来限制哪些单元测试运行，也可以使用 `-test-sdk` 运行在指定的 SDK 版本上。

可以选择在运行单元测试时指定 `-testTimeout`。当某个独立的测试到了超时时间，会被认为失败而不是无限等待，可以防止测试任务因为发生错误而导致死锁。

默认情况下，模拟器启动程序单元测试等待最多 30s，可以使用 `-launch-timeout` 指令来修改超时时间。

### 并行运行单元测试

xctool 可以配置并行运行单元测试，以便更好地利用空闲的多核 CPU。

若要允许多个测试任务同时运行，使用 `-parallelize` 指令：

```bash
path/to/xctool.sh \
	-workspace YourWorkSpace.xcworkspace \
	-scheme YourScheme \
	run-tests -parallelize
```

上面给出的并行，仍限制于最慢的测试任务。比如，两个测试任务（'A' 和 'B'），'B' 因为包含 10 倍多测试需要花费 10 倍的时间来运行，这样上述并行运行帮助并不大。

可以使用 `-logicTestBucketSize` 指令将测试任务分为几个 bucket ：

```bash
path/to/xctool.sh \
	-workspace YourWorkSpace.xcworkspace \
	-scheme YourScheme \
	run-tests -parallelize -logicTestBucketSize 20
```

上面将测试任务分为每个 bucket 20 个测试用例，这些测试将同时运行，如果测试包比其他大得多，这样可以加快整体测试运行速度。

## 可持续集成（CI）

对于可持续集成服务器比如 [Travis CI](https://travis-ci.org/) 或者 [Jenkins](http://jenkins-ci.org/) 来运行单元测试，xctool 是不错的选择。为了在可持续集成环境中运行单元测试，必须为应用的 target 创建一个 **Shared Schemes** target，同时确保所有依赖已经正确添加到 Scheme 中（比如 CocoaPods）。参照如下步骤：

1. 选择 **Product** > **Schemes** > **Manage Schemes** 菜单，打开 **Manage Schemes**。
2. 在列表中找到应用的 target，确保在右手边的 **Shared** 复选框已经选中。
3. 如果应用或者单元测试的 target 包含跨项目的依赖比如 CocoaPods，还需要确保它们已经正确配置。可参照如下步骤：

	* 选中应用的 target，点击 **Edit** 按钮打开 Scheme 编辑弹窗。
	* 在 Scheme 编辑器里面点击左手边的 **Build** 选项。
	* 点击 + 号按钮为项目添加所有依赖，CocoaPods 将以名字为 **Pods** 的静态库出现。
	* 拖动依赖到应用的 target 下方，方便应用的 target 优先编译。

现在，在工程项目里的 **xcshareddata/xcschemes** 目录下会有一个文件，这就是刚刚配置好的 **Shared Scheme**，在 git\svn 仓库中添加该文件，xctool 将在下一次 CI 编译时，找到并执行单元测试。

### Travis CI 配置示例

[Travis CI](https://travis-ci.org/) 是一个非常流行的，免费提供给开源项目的可持续集成系统，它集成于 Github，并且对于 Objective-C 工程，Travis CI 默认使用 xctool 作为编译和测试工具。按照上面的步骤设置好 Shared Schemes 后，还需要配置 `.travis.yml` 文件。

如果是使用 workspace，`.travis.yml` 文件内容格式如下：

```
language: objective-c
xcode_workspace: path/to/YourApp.xcworkspace
xcode_scheme: YourApp
```

如果是使用 projects，`.travis.yml` 文件内容格式如下：

```
language: objective-c
xcode_project: path/to/YourApp.xcodeproj
xcode_scheme: YourApp
```

`.travis.yml` 文件还可以灵活配置怎样安装 Travis 和调用 xctool：

```
language: objective-c
before_install:
    - brew update
    - brew install xctool
script: xctool -workspace MyApp.xcworkspace -scheme MyApp test
```

你可以在 [About OS X Travis CI Environment](http://about.travis-ci.org/docs/user/osx-ci-environment/) 这份文档中学到更多有关 iOS 和 OSX 应用的 Travis CI 环境，通过查阅 [Getting Started](https://docs.travis-ci.com/user/getting-started/) 找到配置项目的详细文档。


## 日志样式

xctool 以不同的样式输出编译构建和测试结果日志，如果不指定日志样式，xctool 默认使用 `pretty` 和 `user-notifications`，xctool 也有如下规则：

* 如果没有检测到 TTY，则不能覆盖 `pretty` 这种日志样式，可以通过在环境中设置 `XCTOOL_FORCE_TTY` 来覆盖。
* 如果 xctool 检测到编译构建任务是在 Travis CI，CircleCI，TeamCity，或者 Jenkins（即在环境中设置 TRAVIS=true，CIRCLECI=true，TEAMCITY_VERSION，或者 JENKINS_URL）上运行的，`user-notifications` 样式是不会使用的。

可以通过 `-option` 选项来选择自己的日志输出样式：

```
path/to/xctool.sh \
  -workspace YourWorkspace.xcworkspace \
  -scheme YourScheme \
  -reporter plain \
  build
```

默认情况下，日志是标准输出，但是也可以直接在日志样式选项后追加 `:OUTPUT_PATH` 来指定日志输出到文件中：

```
path/to/xctool.sh \
  -workspace YourWorkspace.xcworkspace \
  -scheme YourScheme \
  -reporter plain:/path/to/plain-output.txt \
  build
```

只要喜欢，可以使用尽可能多的日志样式，只需多次使用 `-reporter` 选项。

### 包含的日志样式

* **pretty**：一个使用 ANSI 颜色和 Unicode 字符输出漂亮的基于文本的日志样式（默认）。
* **plain**：和 *pretty* 类似，但是没有颜色和 Unicode 字符。
* **phabricator**：将编译构建、测试结果输出成一个 JSON 数组的样式，可以导入到 [Phabricator](http://phabricator.org/) 代码审查工具。
* **junit**：测试结果生成一个 JUnit/xUnit 兼容的 XML 文件。
* **json-stream**：一个以 JSON 字典格式输出编译构建、测试事件流，每行一个（[样式示例](https://gist.github.com/fpotter/82ffcc3d9a49d10ee41b)）。
* **json-compilation-database**：输出编译构建事件的 [JSON Compilation Database](http://clang.llvm.org/docs/JSONCompilationDatabase.html)，被用于 [Clang Tooling](http://clang.llvm.org/docs/LibTooling.html)，例如 [OCLint](http://oclint.org/)。
* **user-notifications**：当事件\任务完成后，会给通知中心发送通知（[示例通知](https://cloud.githubusercontent.com/assets/1044236/2771974/a2715306-ca74-11e3-9889-fa50607cc412.png)）。
* **teamcity**：发送服务消息到 [TeamCity]() 持续集成服务器。

### 实现自己的日志样式

你也可以使用任何你喜欢的语言来实现自己的日志样式，在 xctool 中，日志样式是独立可执行的，从 STDIN 读取 JSON 对象，格式化结果写入到 STDOUT。

你可以通过 `-reporter` 选项指定完整路径调用自定义日志样式：

```
path/to/xctool.sh \
  -workspace YourWorkspace.xcworkspace \
  -scheme YourScheme \
  -reporter /path/to/your/reporter \
  test
```

例如，下面是用 Python 写的一个简单的日志样式，每次测试通过输出一个句号 `.`，失败则输出一个感叹号 `!`。

```python
#!/usr/bin/python

import fileinput
import json
import sys

for line in fileinput.input():
    obj = json.loads(line)

    if obj['event'] == 'end-test':
        if obj['succeeded']:
            sys.stdout.write('.')
        else:
            sys.stdout.write('!')

sys.stdout.write('\n')
```

如果你用 Objective-C 写日志样式，你会发现 `Reporter` 类比较有帮助，见 [Reporter.h]()。

## 配置（.xctool-args）

如果你在命令行中经常需要通过很多参数来使用 *xctool*，可以设置 **.xctool-args** 文件来加快工作流程。如果 *xctool* 在当前目录中发现 **.xctool-args** 文件，它会自动在参数那里预填充。

格式是一个 JSON 数组的参数配置：

```json
[
  "-workspace", "YourWorkspace.xcworkspace",
  "-scheme", "YourScheme",
  "-configuration", "Debug",
  "-sdk", "iphonesimulator",
  "-arch", "i386"
]
```

通过命令行配置的任何额外参数优先于它们在 *.xctool-args* 文件中的配置。

