---
layout: post
title: "自定义转场动画介绍(翻译)"
date: 2015-05-30 21:37:02 +0800
comments: true
categories: 
---

[英文原文地址](http://www.appcoda.com/custom-view-controller-transitions-tutorial/)

仔细观察 iOS 设备上苹果内置的 app，你会发现各种视图控制器转场动画。iOS7 介绍了自定义控制器转场，使开发人员能够在他们的应用中创建自己的转场动画。在本教程中，我们将看到如何做到这一点。我们还将了解怎样通过手势进行交互式转场 `interactive transitions`。

<!--more-->

![](http://i1.tietuku.com/41b836336adda5e8.jpg)


## 开始

创建自定义转场必须遵循以下三个步骤：


1. 创建一个类，遵守 `UIViewControllerAnimatedTransitioning` 协议，并实现协议中的方法。在这个类中，将编写执行动画的代码，这个类被用做动画控制器。
2. 在 `present` 控制器前，需设置一个类作为其转场代理。这个代理用于当 `present` 控制器时，从动画控制器中回调。
3. 实现回调方法，返回一个第一步中的动画控制器实例对象。


运行初始项目你会看到一组 items 的 tableView，在导航栏右边有一个 `Action` 按钮，当你点击它，会以默认 `modal` 样式从底部 `present` 出一个控制器，我们将为这个视图转场编写自定义转场。

![](http://i1.tietuku.com/86214a9a01cf1d06.gif)

## 自定义 Present 转场

如前所述，接下来做的第一件事是创建动画控制器。创建一个 `CustomPresentAnimationController` 类继承自 `NSObject`，并遵守 `UIViewControllerAnimatedTransitioning` 协议。

```swift
class CustomPresentAnimationController: NSObject, UIViewControllerAnimatedTransitioning {
}
```

`UIViewControllerAnimatedTransitioning` 协议有两个必须实现的方法，在这个类中添加以下代码。

```swift
/**
动画持续时间
*/
func transitionDuration(transitionContext: UIViewControllerContextTransitioning) -> NSTimeInterval {
    return 2.5
}

/**
转场动画
*/
func animateTransition(transitionContext: UIViewControllerContextTransitioning) {
    
    let fromViewController = transitionContext.viewControllerForKey(UITransitionContextFromViewControllerKey)!
    let toViewController = transitionContext.viewControllerForKey(UITransitionContextToViewControllerKey)!
    let finalFrameForVC = transitionContext.finalFrameForViewController(toViewController)
    let containerView = transitionContext.containerView()
    let bounds = UIScreen.mainScreen().bounds
    
    // 设置目标控制器的 frame
    toViewController.view.frame = CGRectOffset(finalFrameForVC, 0, bounds.height)
    // 将目标控制器的根 view 添加至容器视图 containerView
    containerView.addSubview(toViewController.view)
    
    // 设置动画代码
    UIView.animateWithDuration(transitionDuration(transitionContext), delay: 0.0, usingSpringWithDamping: 0.5, initialSpringVelocity: 0, options: UIViewAnimationOptions.CurveLinear, animations: { () -> Void in
        
        fromViewController.view.alpha = 0.5
        toViewController.view.frame = finalFrameForVC
        
    }) { (_) in
        
        // 动画结束后 通知转场上下文转场结束
        transitionContext.completeTransition(true)
        fromViewController.view.alpha = 1.0
    }
}
```

第一方法是设置转场*动画时长*，这个 Demo 中我们设置2.5秒，但是在实际应用中，你应该设置一个比这个数小的数。

第二个方法，我们通过转场上下文 `transitionContext` 取得*来源控制器* fromVC、*目标控制器* toVC、动画结束后终止 frame，以及给 fromVC 和 toVC 转场切换视图提供容器的*容器视图* `containerView`。

接下来我们设置 toView 在屏幕的底部，然后将 toView 添加至容器视图和动画的闭包中，让 toView 动画至设定的最终位置。同时也设置了 fromView 的透明度动画，以至于随着 toView 的向上滑动 fromView 逐渐淡出。动画时间通过 `transitionDuration(transitionContext)` 获得。在完成闭包中，我们通知转场上下文转场结束，接着让 fromView 的透明度恢复正常。容器视图将会移除 fromView。

动画控制器类里面的任务已经完成，下一步我们需要将它和 `storyboard segue` 连接起来。

打开 ItemsTableViewController.swift 文件，遵守 `UIViewControllerTransitioningDelegate` 协议。

```swift
class ItemsTableViewController: UITableViewController, UIViewControllerTransitioningDelegate {
}
```

UIViewController 有一个 `transitionDelegate` 属性，支持自定义转场。当转场至一个控制器时，会通过这个属性查看自定义转场是否使用。`UIViewControllerTransitioningDelegate` 提供自定义转场。

打开 Main.storyboard 文件，选择 `Present modally segue to Action View Controller`，在属性检查器中，设置它的 Identifier 为 `showAction`。

![](http://i1.tietuku.com/f710e825b83b99be.png)

回到 ItemsTableViewController.swift 文件，添加如下代码。

```swift
/// 实例化一个 Present 动画控制器
let customPresentAnimationController = CustomPresentAnimationController()

override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
    
    if  segue.identifier == "showAction" {
        let toViewController = segue.destinationViewController as! UIViewController
        // 设置目标控制器的转场代理为当前控制器
        toViewController.transitioningDelegate = self
    }
}
```

在这里我们实例化一个 Present 动画控制器，在 `prepareForSegue()` 函数中，设置目标控制器的 `transitioningDelegate` 属性。

继续添加如下代码，返回自定义 `Present` 动画控制器实例对象。

```swift
/**
返回自定义 Present 动画对象
*/
func animationControllerForPresentedController(presented: UIViewController, presentingController presenting: UIViewController, sourceController source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
    return customPresentAnimationController
}
```

运行程序，你会看到下图效果，Action view 在屏幕底部向上缓慢滑动，停止前会有小的弹簧效果。

![](http://i1.tietuku.com/ca3edd567594f674.gif)

在 CustomPresentAnimationController.swift 更改代码如下，你会看到一个稍微不同的效果，目标控制器的初始位置是在屏幕上方。

```swift
toViewController.view.frame = CGRectOffset(finalFrameForVC, 0, -bounds.height)
```

运行程序，这次 Action view 是从屏幕顶部缓慢落下。

## 自定义 Dismiss 转场

我们已经实现了自定义 `present` 转场动画，但是 `dismiss` 转场动画还是使用的苹果默认的。

`UIViewControllerTransitioningDelegate` 协议允许同时设置负责 `dismiss` 和 `present` 控制器的转场动画控制器。

创建一个 `CustomDismissAnimationController` 类，继承自 `NSObject`，并遵守 `UIViewControllerAnimatedTransitioning` 协议。

```swift
class CustomDismissAnimationController: NSObject, UIViewControllerAnimatedTransitioning {
}
```

在这个类添加如下代码。

```swift
/**
动画持续时间
*/
func transitionDuration(transitionContext: UIViewControllerContextTransitioning) -> NSTimeInterval {
    return 2.0
}

/**
转场动画
*/
func animateTransition(transitionContext: UIViewControllerContextTransitioning) {
    
    let fromViewController = transitionContext.viewControllerForKey(UITransitionContextFromViewControllerKey)!
    let toViewController = transitionContext.viewControllerForKey(UITransitionContextToViewControllerKey)!
    let finalFrameForVC = transitionContext.finalFrameForViewController(toViewController)
    let containerView = transitionContext.containerView()
    
    // 设置 toVC 的根 view 的 frame 以及透明度
    toViewController.view.frame = finalFrameForVC
    toViewController.view.alpha = 0.5
    // 将目标控制器的根 view 添加至容器视图 containerView
    containerView.addSubview(toViewController.view)
    containerView.sendSubviewToBack(toViewController.view)
    
    // 设置动画代码
    UIView.animateWithDuration(transitionDuration(transitionContext), animations: { () -> Void in
        
        fromViewController.view.frame = CGRectInset(fromViewController.view.frame, fromViewController.view.frame.width / 2, fromViewController.view.frame.height / 2)
        toViewController.view.alpha = 1
        
    }) { (_) -> Void in
        
        // 动画结束后 通知转场上下文转场结束
        transitionContext.completeTransition(true)
    }
```

这段代码和 `present` 转场动画代码实现方式很相似。在 `animateTransition()` 函数中，拿到当前转场上下文的 toVC 和 fromVC。现在的 toVC 是表视图控制器，我们设置了它的根视图的 alpha 值，动画开始后，将逐渐淡入。然后将 toVC 的根 view 添加至容器视图，并放在 fromVC 的根 view 的后面。

在动画闭包中，设置了 fromVC 的根 view 的宽和高动画到0，并保持在屏幕的中心，这会产生一个 from view 缩小到消失的动画效果，同时 to view 逐渐可见。

在 ItemsTableViewController.swift 文件中添加下面这个属性。

```swift
/// 实例化一个负责 Dismiss 转场动画控制器
let customDismissAnimationController = CustomDismissAnimationController()
```

同时实现下面这个代理方法。

```swift
/**
返回负责自定义 Dismiss 转场动画对象
*/
func animationControllerForDismissedController(dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
    return customDismissAnimationController
}
```

`UIViewControllerTransitioningDelegate` 协议提供了上面这个函数，可以设置负责 dismiss 转场动画的控制器。

运行程序，你会看到下面的动画。

![](http://i1.tietuku.com/b42fc6619b716079.gif)

显然，这并不是我们想要的动画。from view 的消失动画是我们想要的，但是图片 image 的相对位置没有变化。这是因为，我们改变的是 from view 的 frame，这并不会影响到它的子视图。我们将用 UIView 的快照 `snapshot` 来修复这个 bug。

UIView 快照是对当前存在的 UIView 截图让它成为一个轻量级的 UIView，我们将在动画中使用快照而不是真正的 view。

修改 `animateTransition()` 函数内部如下。

```swift
func animateTransition(transitionContext: UIViewControllerContextTransitioning) {
    
    let fromViewController = transitionContext.viewControllerForKey(UITransitionContextFromViewControllerKey)!
    let toViewController = transitionContext.viewControllerForKey(UITransitionContextToViewControllerKey)!
    let finalFrameForVC = transitionContext.finalFrameForViewController(toViewController)
    let containerView = transitionContext.containerView()
    
    // 设置 toVC 的根 view 的 frame 以及透明度
    toViewController.view.frame = finalFrameForVC
    toViewController.view.alpha = 0.5
    // 将目标控制器的根 view 添加至容器视图 containerView
    containerView.addSubview(toViewController.view)
    containerView.sendSubviewToBack(toViewController.view)
    
    // 生成快照 并添加到容器视图
    let snapshotView = fromViewController.view.snapshotViewAfterScreenUpdates(false)
    snapshotView.frame = fromViewController.view.frame
    containerView.addSubview(snapshotView)
    
    // 移除 fromVC 的根 view
    fromViewController.view.removeFromSuperview()
    
    // 设置动画代码
    UIView.animateWithDuration(transitionDuration(transitionContext), animations: { () -> Void in
        
        snapshotView.frame = CGRectInset(fromViewController.view.frame, fromViewController.view.frame.width / 2, fromViewController.view.frame.height / 2)
        toViewController.view.alpha = 1
        
    }) { (_) -> Void in
        
        // 动画结束后 通知转场上下文转场结束 并移除快照
        snapshotView.removeFromSuperview()
        transitionContext.completeTransition(true)
    }
}
```

这里我们创建了一个 fromVC 的根 view 的快照，添加到容器视图，并移除 fromVC 的根 view。转场动画中缩放快照，动画完成后将快照从容器视图中移除。

运行程序，转场动画一切正常。

![](http://i1.tietuku.com/c7d1a6eb3c4a7645.gif)


