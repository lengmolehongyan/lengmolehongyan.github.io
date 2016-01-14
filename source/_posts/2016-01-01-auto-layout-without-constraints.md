---
layout: post
title: "没有约束的自动布局（Auto Layout Without Constraints）"
date: 2016-01-01 10:38:41 +0800
comments: true
categories: 
keywords: Auto Layout Without Constraints，没有约束的自动布局
description: 没有约束的自动布局（Auto Layout Without Constraints）
---

StackView 提供了一种简单的方法，剔除了复杂的约束，利用自动布局的强大来布局界面，单个 StackView 由一行或者一列控件组成，StackView 根据自身的属性布局这些控件。

* [axis](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIStackView_Class_Reference/index.html#//apple_ref/occ/instp/UIStackView/axis):（仅 UIStackView）表示 StackView 的方向，纵向或者水平。
* orientation:（仅 NSStackView）表示 StackView 的方向，纵向或者水平。
* [distribution](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIStackView_Class_Reference/index.html#//apple_ref/occ/instp/UIStackView/distribution)：表示沿着 axis 方向的视图布局方式。
* [alignment](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIStackView_Class_Reference/index.html#//apple_ref/occ/instp/UIStackView/alignment)：表示纵向于 axis 方向的视图布局方式。
* [spacing](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIStackView_Class_Reference/index.html#//apple_ref/occ/instp/UIStackView/spacing)：表示相邻视图之间的间隔。

<p></p>
要使用 StackView，在 IB 中拖一个纵向或者水平的 StackView 到画布中，然后拖控件放到 StackView 内。

如果一个视图有自身内容大小，它在 StackView 中将显示为自身的大小。如果没有，IB 将会提供默认的大小，你可以调整它的大小，IB 中添加约束来保持大小不变。

想进一步微调布局，可以在属性检查器（Attributes Inspector） 修改 StackView 的属性。下面这个例子，设置 StackView 的 spacing 属性为 8 个点，distribution 属性为 Fill Equally。

<!--more-->

![IB_StackView_Simple](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/Art/IB_StackView_Simple_2x.png)

StackView 的布局还基于子视图的内容压缩和抗压缩优先级，可以在尺寸检查器（Size Inspector）修改。

> 注：
> 可以通过给子视图直接添加约束来更进一步的调整布局，但是，要避免任何可能的冲突。通用法则：如果视图对于一个给定的大小默认回到其本身内容大小，便可以安全添加该约束。更多约束冲突，参阅 [约束冲突（Unsatisfiable Layouts）]()。

此外，你还可以用 StackView 嵌套 StackView，构建更加复杂的布局。

![IB_StackView_NestedStacks](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/Art/IB_StackView_NestedStacks_2x.png)

一般情况下，尽可能多的使用 StackView 来管理布局，只有使用 StackView 不能达到效果时，再去创建约束。

有关 StackView 的更多使用，参阅 [UIStackView Class Reference](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIStackView_Class_Reference/index.html#//apple_ref/doc/uid/TP40015256) 或者 NSStackView Class Reference。

> 注：
> 虽然嵌套使用 StackView 可以构建出复杂的界面，但仍不能完全避开约束的使用，因为至少总是需要设置最外层视图的位置（和可能的大小）。

原文链接：[Auto Layout Without Constraints](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/AutoLayoutWithoutConstraints.html#//apple_ref/doc/uid/TP40010853-CH8-SW1)


