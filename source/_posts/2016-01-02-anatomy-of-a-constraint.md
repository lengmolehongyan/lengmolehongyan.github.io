---
layout: post
title: "约束剖析（Anatomy of a Constraint）"
date: 2016-01-02 12:09:52 +0800
comments: true
categories: 
keywords: iOS，约束剖析，自动布局
description: iOS，约束剖析，自动布局
---

视图层次结构上的布局是由一些列的公式（线性方程）来定义的，每个约束都有一个单一的表达式，你的目标就是要让这一系列的表达式有且仅有一个结果。

示例公式如下：

<!--more-->

![view_formula](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/Art/view_formula_2x.png)

这个约束声明了红色视图（view）的左边（leading）距离蓝色视图（view）的右边（trailing）始终保持 8 个点，这个公式有如下几项：

* **item 1（第一项）**：表达式中的第一项，在这个例子中，就是红色视图（view）。这一项必须是一个视图（view）或者是一个布局参考线（layout guide）。
* **Attribute 1（属性 1）**：要约束表达式中第一项的属性，在此例中，就是红色视图（view）的左边（leading）。
* **Relationship（关系）**：表达式左右两边之间的关系，有三种关系，等于，大于等于，小于等于，在此例中，左右两边是相等的关系。
* **Multiplier（乘数）**：此浮点数值将用于和属性 2 的值相乘，在此例中，这个值是 1.0。
* **Item 2（第二项）**：表达式中的第二项，在此例中就是蓝色视图（view），不像第一项，这一项可以是空的。
* **Attribute 2（属性 2）**：要约束表达式中第二项的属性，在此例中，就是蓝色视图（view）的右边（trailing）。如果第二项是空，这里一定不是一个属性。
* **Constant（常量）**：一个表示偏移量的浮点数常量，在此例中，值是 8.0。这个值是用来添加到属性 2 上的。

<p></p>
界面上的大多数约束是由两项之间的关系表示的，这些项可以是视图，也可以是布局参考线。约束也可以表示同一视图两个不同属性之间的关系，比如，设置视图的宽高比例，也可以给视图的高或宽设置为一个常量值，当设置为常量值时，第二项为空，第二项的属性设置为 Not An Attribute，乘数为 0.0。

## 自动布局属性

在自动布局中，属性的功能就是被约束的，通常包括四个边界（前后上下），还有高度、宽度以及垂直和水平中心。文本控件还有一个或多个基线（baseline）属性。

![attributes](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/Art/attributes_2x.png)

有关完整的布局属性列表，参阅 [NSLayoutAttribute 枚举](https://developer.apple.com/library/ios/documentation/AppKit/Reference/NSLayoutConstraint_Class/index.html#//apple_ref/c/tdef/NSLayoutAttribute) 。

> 注：
> 虽然 OSX 和 iOS 都是使用 NSLayoutAttribute 枚举，但它们定义的值集稍有不同，查阅完整布局属性列表时，请确保查看的是正确平台的文档。


## 示例方程

通过方程中大量的参数和属性可以创建多种不同类型的约束，你可以定义两个视图之间的间距、视图边缘的对齐方式、两个视图的相对大小，甚至定义视图的纵横比。然而，并非所有属性都是兼容的。

有两种基本类型的属性，大小属性（例如，高度和宽度）和位置属性（例如，左边和顶部）。大小属性用于指定一个视图有多大，并不指定视图的位置。位置属性用于指定视图相对其他视图的位置，同样不涉及视图的大小。

带着这些差异，再看看下面这些规则：

* 大小属性的约束不能添加到位置属性上
* 大小属性只能赋值常量
* 对于位置属性，不能将垂直属性的约束添加到水平属性上
* 对于位置属性，不能将 Leading 和 Trailing 属性的约束添加到 Left 和 Right 属性上

<p></p>
例如，没有额外的属性仅设置一个视图的顶部为一个常量值 20.0 是没有意义的，必须总是定义某一视图相对于其他视图的位置属性，比如，设置距离其父视图的顶部为 20.0 个点。然而，设置某一视图的高度是 20.0 是有效的。更多信息，参阅 [自动布局属性值解释]()。

3-1 列出了不同常见约束的表达式。

> 注：
> 本章节所有示例方程都是使用的伪代码，想看实际代码创建约束，参阅 [自动布局手册]() 和 [代码创建约束]()。

```objc
// 3-1 常见约束示例表达式
// 设置高度常量值
View.height = 0.0 * NotAnAttribute + 40.0
 
// 设置两个按钮之间的固定间距
Button_2.leading = 1.0 * Button_1.trailing + 8.0
 
// 设置两个按钮的前部边缘对齐
Button_1.leading = 1.0 * Button_2.leading + 0.0
 
// 设置两个按钮的宽度相同
Button_1.width = 1.0 * Button_2.width + 0.0
 
// 设置视图在其父视图的中心
View.centerX = 1.0 * Superview.centerX + 0.0
View.centerY = 1.0 * Superview.centerY + 0.0
 
// 设置视图的宽高比为一个常量值
View.height = 2.0 * View.width + 0.0
```

## 相等，不是赋值

注意，示例中方程的等号表示的是相等，而不是赋值。

当自动布局求解这些方程时，并不是将等式右边的值赋值给等式左边。相反，它同时计算属性 1 和属性 2 的值使它们之间的关系成立，这就意味着可以自由调整表达式中的项。比如，3-2 中的方程就和 3-1 的结果是相同的。

```objc
// 3-2 反向方程
// 设置两个按钮之间的固定间距
Button_1.trailing = 1.0 * Button_2.leading - 8.0
 
// 设置两个按钮的前部边缘对齐
Button_2.leading = 1.0 * Button_1.leading + 0.0
 
// 设置两个按钮宽度相同
Button_2.width = 1.0 * Button.width + 0.0
 
// 设置视图在其父视图的中心
Superview.centerX = 1.0 * View.centerX + 0.0
Superview.centerY = 1.0 * View.centerY + 0.0
 
// 设置视图的宽高比为一个常量值
View.width = 0.5 * View.height + 0.0
```

> 注：
> 当重新调整方程中的项时，请确保调整了乘数和常量。比如，常量值 8.0 要变成 -8.0，乘数 2.0 要变成 0.5，常量值 0.0 和 乘数 1.0 保持不变。

你会发现，自动布局经常提供多种方式来解决同样的问题。理想情况下，你应该选择的是最能清晰表达意图的方案。然而，不同的开发者无疑会不同意哪个方案是最好的。这里，相比正确的方案，一贯选择好的方案。如果你选择一种方案，并坚持使用，从长远来看，你将遇到很少问题。比如，本指南使用以下规则：

1. 整数乘法优先于小数乘法。
2. 正数优先于负数。
3. 不管在哪，视图应该以从前到后，从上到下的布局顺序。

## 创建没有歧义的、不冲突的布局

当使用自动布局时，我们的目标就是，提供一系列方程让它们有且仅有一种可能的解决方案。有歧义的约束有多个可能的解决方案，冲突的约束不会有有效的解决方案。

一般来说，须为每个视图都指定大小和位置的约束。但是，如果父视图的大小已经设置（比如，在 iOS 中控制器的根视图），没有歧义的、不冲突的布局需要为每个视图的每个维度的设置两个约束条件（不包括父视图）。然而，在选择你想要使用的约束的时有大量的选择。例如，下面的三种方式都可以创建出没有歧义的、不冲突的布局（只显示了水平方向的约束）：

![constraint_examples](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/Art/constraint_examples_2x.png)

* 第一种布局约束了视图的左边缘相对于父视图的左边缘的位置，同时设置了该视图的宽度，右边缘的位置可以基于父视图的大小以及其他约束计算出来。
* 第二种布局约束了视图的左边缘相对于父视图的左边缘的位置，同时约束了视图的右边缘相对于父视图的右边缘的位置，该视图的宽度可以基于父视图的大小以及其他约束计算出来。
* 第三种布局约束了视图的左边缘相对于父视图的左边缘的位置，同时该视图和父视图 center 中心对齐，视图的大小和距离父视图右边缘的位置都可以基于父视图的大小以及其他约束计算出来。

注意每种布局都有一个视图和两个水平方向的约束。在每种情况下，约束完全定义了视图的宽度和水平位置。这意味着所有的布局沿着水平轴创建了没有歧义的，不冲突的布局。然而这些布局并不完全相同，想象一下，如果父视图的宽度改变了将会发生什么？

在第一种布局中，视图的宽度不会发生改变。大多数时候，这并不是你所想要的。事实上，作为一般原则，你应该避免给视图的大小赋值常量。自动布局是设计给创建布局来动态适应不同的环境。当给视图设置一个固定大小，就削弱了自动布局的强大了。

第二种布局和第三种布局产生相同的效果，可能并不明显：父视图的宽度改变了，它们都和父视图保持一个固定的间距。然而，它们并不一定相同。一般来说，第二个例子更容易理解，但是第三个例子可能是更有用的，尤其是设置一些视图 center 中心对齐。一如既往，为你的特点布局选择最佳的方法。

现在考虑一些更复杂的情况，假设你想在一个 iPhone 上显示两个视图，一边一个，你需要确保它们在各边界上有个好的间距，且总是有相同的宽度，而且在设备选中的时候，它们也能正确的做出调整。

下面的附图展示了在竖屏、横屏情况下的两个视图：

![Blocks_Portrait](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/Art/Blocks_Portrait_2x.png)

![Blocks_Landscape](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/Art/Blocks_Landscape_2x.png)

所以，这样的约束应该是什么样子呢？下面的附图提供了一种解决方案：

![two_view_example_1](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/Art/two_view_example_1_2x.png)

上面的解决方案使用了下面这些约束：

```objc
// 垂直约束
Red.top = 1.0 * Superview.top + 20.0
Superview.bottom = 1.0 * Red.bottom + 20.0
Blue.top = 1.0 * Superview.top + 20.0
Superview.bottom = 1.0 * Blue.bottom + 20.0
 
// 水平约束
Red.leading = 1.0 * Superview.leading + 20.0
Blue.leading = 1.0 * Red.trailing + 8.0
Superview.trailing = 1.0 * Blue.trailing + 20.0
Red.width = 1.0 * Blue.width + 0.0
```

按照之前的经验法则，这种布局有两个视图，四个水平约束和四个垂直约束。虽然这不是一个绝对有效的法则，但它可以表示出你已经在正确的道路上。更重要的是，约束唯一指定了每个视图的大小和位置，生成一个没有歧义的、不冲突的布局。移除任何一个约束，布局就会变成有歧义的，添加额外的约束，就会导致约束冲突。

当然，这并不是唯一的解决方案。这有一个同样有效的方法：

![two_view_example_2](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/Art/two_view_example_2_2x.png)

代替固定蓝色视图的顶部和底部距离父视图的位置，让蓝色视图和红色视图顶部对齐，同时让蓝色视图和红色视图底部对齐，约束如下所示：

```objc
// 垂直约束
Red.top = 1.0 * Superview.top + 20.0
Superview.bottom = 1.0 * Red.bottom + 20.0
Red.top = 1.0 * Blue.top + 0.0
Red.bottom = 1.0 * Blue.bottom + 0.0
 
// 水平约束
Red.leading = 1.0 * Superview.leading + 20.0
Blue.leading = 1.0 * Red.trailing + 8.0
Superview.trailing = 1.0 * Blue.trailing + 20.0
Red.width = 1.0 * Blue.width + 0.0
```

该例子中任然有两个视图，四个水平约束，四个垂直约束，仍然会产生没有歧义的、不冲突的布局。

> 但是哪个更好呢？
> 这些方案都会产生有效的布局，因此哪个更好呢？
> 不幸的是，客观证明一种方法优于其他方法是几乎不可能的，因为每个都有自己的优点和缺点。




