---
layout: post
title: "在Xcode6中使用IBDesignable创建自定义控件(翻译)"
date: 2015-05-28 00:16:31 +0800
comments: true
category: 
---

[英文原文地址](http://www.appcoda.com/ibdesignable-ibinspectable-tutorial/)

在Xcode的旧版本中，试图创建一个自定义控件，并不是很容易，因为在IB中，并不能实时预览到你的设计成果，只能在模拟器中测试。对于设计一个单一组件，可能需要花费大量时间。

Xcode6的发布，苹果为开发者构建自定义控件推出了新功能**IBDesignable**和**IBInspectable**，允许在IB中实时预览设计成果。很明显，这会给我们在实际开发过程中提高不少效率。

在本教程中，将介绍IBDesignable IBInspectable，以及展示如何利用这个新功能。除过创建demo示例没有更好地方式来阐述这一新特性，因此，创建一个"Rainbow"自定义界面。

![效果图](http://i1.tietuku.com/a476b2f40c2465d0.png)

***

<!--more-->

### IBDesignable和IBInspectable

使用IBDesignable和IBInspectable，开发者创建界面(或视图)可以实时呈现在IB中。一般来说，为了使用这个新特性，你需要做的是创建一个`UIView`或者`UIControl`的子类，然后在定义类的前面加上`@IBDesignable`关键字。如果是OC，使用`IB_DESIGNABLE`宏。下面是Swift示例代码:

```swift
@IBDesignable
class Rainbow: UIView {
}
```

在Xcode旧版本中，你可以在IB中编辑`User Defined Runtime Attributes`来改变一个对象的属性(例如:layer.cornerRadius)，问题是你需要确切知道属性名。`IBInspectable`只需要一步，对一个可视化类的属性前面加上`IBInspectable`关键字前缀，该属性会在暴露在IB中，这就是一个更改属性值更简单的方法。

![IB属性](http://i1.tietuku.com/93c53c69c11bffa1.png)

你如果使用Swift开发app，你需要做的只是在你选择的属性前面加上`@IBInspectable`关键字，下面是个示例代码片段:

```swift
@IBInspectable var firstColor: UIColor = UIColor.blackColor() {
   // 值改变时更新UI
}
```

### 创建Xcode项目

创建一个新的Xcode项目，选择Single View Application模板，起名为*RainbowDemo*，在此项目中，将使用Swift语言，因此，创建项目时不要忘记勾选。

完成后，选择Main.storyboard文件，设置View Controller的根视图View的背景颜色Hex Color值为`38334C`(或者任何你想要的颜色)。然后从对象库中拖一个View放进View Controller，设置它的大小Width为600，Height为434，然后把它放在根视图的中心，设置新视图View和根视图相同背景颜色。

> 提示：如果想改变RGB颜色值，只需打开调色板和切换到滑块标签来改变RGB值

![设置背景颜色](http://i1.tietuku.com/ae13d9f742d52315.gif)

在Xcode6中，为了适配各个iOS设备，你必须为视图View配置自动布局约束。对于简单的约束，你可以在自动布局菜单单击**Issues**选项，选择`Add Missing Contraints`，Xcode将自动为View添加布局约束。

![添加约束](http://i1.tietuku.com/7b993f7b47a011c2.gif)

***

### 创建自定义View类

现在，你已经在storyboard中创建了一个View，是时候创建自定义View类了。使用Cocoa Touch Class文件模板，创建自定义类文件，继承自UIView，起名为"Rainbow"。

![创建自定义View类](http://i1.tietuku.com/08e097c646676e6a.png)

在自定义类中插入以下代码:

```swift
import UIKit

class Rainbow: UIView {
    override init(frame: CGRect) {
        super.init(frame: frame)
    }
    required init(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
    }
}
```

如前所述，这个可视化类是UIView的子类，让自定义类实时呈现，需要`override`上述两个方法。然后，打开辅助编辑器，更改新拖的View的自定义类为*Rainbow*类。

![](http://i1.tietuku.com/5e41283a3fd5d23c.gif)

***

### 实现IBDesignable控制

为了实现实时预览，在自定义类前面加一个前缀`@IBDesignable`关键字

```swift
@IBDesignable 
class Rainbow: UIView {
    ...
}
```

这个关键字确实简单，但是这简单地关键字将使你的开发更加容易。接下来，添加一些设置颜色的属性。在Rainbow类中插入以下代码:

```swift
@IBInspectable var firstColor: UIColor = UIColor(red: (37.0/255.0), green: (252.0/255), blue: (244.0/255.0), alpha: 1.0)
@IBInspectable var secondColor: UIColor = UIColor(red: (171.0/255.0), green: (250.0/255), blue: (81.0/255.0), alpha: 1.0)
@IBInspectable var thirdColor: UIColor = UIColor(red: (238.0/255.0), green: (32.0/255), blue: (53.0/255.0), alpha: 1.0)
```

在这里，我们预先定义每个属性一个默认颜色，每次用户更改它们的值时会重绘视图。更重要的是，我们为每个属性加了一个`@IBInspectable`关键字前缀，现在去IB的属性检查器，你可以直观地发现这些属性:

![IB中的属性](http://i1.tietuku.com/49ce99f08cfba0ab.png)

很酷，对吧？IBInspectable通过指示属性，你可以使用颜色选择器可视化地编辑它们。

在Rainbow类中，为了在屏幕上画一个圆，插入以下代码:

```swift
func addOval(lineWidth: CGFloat, path: CGPathRef, strokeStart: CGFloat, strokeEnd: CGFloat, strokeColor: UIColor, fillColor: UIColor, shadowRadius: CGFloat, shadowOpacity: Float, shadowOffset: CGSize) {
        
   let arc = CAShapeLayer()
   arc.lineWidth = lineWidth
   arc.path = path
   arc.strokeStart = strokeStart
   arc.strokeEnd = strokeEnd
   arc.strokeColor = strokeColor.CGColor
   arc.fillColor = fillColor.CGColor
   arc.shadowRadius = shadowRadius
   arc.shadowOpacity = shadowOpacity
   arc.shadowOffset = shadowOffset
   layer.addSublayer(arc)
}
```

为了保证代码的简洁和可读性，我们定义了依据方法调用者传入参数来绘制一个完整的圆或者半圆的公共方法。利用CAShapeLayer类可以很简单的画一个圆或圆弧。你可以使用strokeStart和strokeEnd属性控制渲染的开始和结束。通过改变strokeEnd的值在0.0到1.0之间，你可以绘制一个完整或者部分的圆。其余的属性是只是用于设置渲染颜色，阴影颜色等，在CAShaperLayer官方文档中可以查看更详细的所有可用属性。

接下来，添加以下方法:

```swift
override func drawRect(rect: CGRect) {
    // 添加圆弧
    addCircle(80, capRadius: 20, color: firstColor)
    addCircle(150, capRadius: 20, color: secondColor)
    addCircle(215, capRadius: 20, color: thirdColor)
}

func addCircle(arcRadius: CGFloat, capRadius: CGFloat, color: UIColor) {
    let x = CGRectGetMidX(bounds)
    let y = CGRectGetMidY(bounds)
    
    // 底部圆弧
    let pathBottom = UIBezierPath(ovalInRect: CGRectMake((x - (arcRadius/2)),
        (y - (arcRadius/2)), arcRadius, arcRadius)).CGPath
    addOval(20.0, path: pathBottom, strokeStart: 0, strokeEnd: 0.5,
        strokeColor: color, fillColor: UIColor.clearColor(),
        shadowRadius: 0, shadowOpacity: 0, shadowOffset: CGSizeZero)
    
    // 中间圆弧
    let pathMiddle = UIBezierPath(ovalInRect: CGRectMake((x - (capRadius/2)) - (arcRadius/2),
        (y - (capRadius/2)), capRadius, capRadius)).CGPath
    addOval(0.0, path: pathMiddle, strokeStart: 0, strokeEnd: 1.0,
        strokeColor: color, fillColor: color,
        shadowRadius: 5.0, shadowOpacity: 0.5, shadowOffset: CGSizeZero)
    
    // 顶部圆弧
    let pathTop = UIBezierPath(ovalInRect: CGRectMake((x - (arcRadius/2)),
        (y - (arcRadius/2)), arcRadius, arcRadius)).CGPath
    addOval(20.0, path: pathTop, strokeStart: 0.5, strokeEnd: 1.0,
        strokeColor: color, fillColor: UIColor.clearColor(),
        shadowRadius: 0, shadowOpacity: 0, shadowOffset: CGSizeZero)
}
```

`drawRect:`方法默认什么也不做，为了在自定义View中画圆，我们override此方法来实现自己的绘制代码。`addCircle:`方法有三个参数：arcRadius，capRadius和color。arcRadius是圆弧的半径，capRadius是圆弧边缘半径。

`addCircle:`方法利用UIBezierPath画圆弧的简单工作原理:

1. 首先，在底部画了个半圆弧
2. 接下来，在圆弧边缘画了一个完整的小圆
3. 最后，画了另一半圆弧

在`drawRect:`方法中，我们调用了`addCircle:`方法三次，传入的参数指定圆弧该怎样画：

![画圆弧原理](http://i1.tietuku.com/c5769f1b0ed47bd7.jpg)

利用IBInspectable属性，你可以在IB中自由改变每个圆弧的颜色，而不需要写代码:

![](http://i1.tietuku.com/dac4b07c68953a6d.png)

显然，你可以进一步利用`@IBInspectable`暴露`arcRadius`属性，便可以在IB中修改绘制圆弧半径。

![修改半径](http://i1.tietuku.com/79c3c33c22943103.jpg)

***

### 总结

通过本教程后，你现在了解了在Xcode6中如何利用IBDesignable和IBInspectable实时预览界面。利用这个新特性，你可以更高效创建自定义组件。

[RainbowDemo地址](https://github.com/lengmolehongyan/RainbowDemo)


