---
layout: post
title: "Can't add self as subview 崩溃解决办法"
date: 2015-10-30 23:14:49 +0800
comments: true
categories: 
keywords: Can't add self as subview
description: Can't add self as subview 崩溃解决办法
---

## 前言

查看项目的崩溃汇总，一直有一个 bug 存在，之前也有注意过，只因这个 bug 的崩溃堆栈日志解析出来并没有什么卵用，也不知该怎么复现，就撂下了。

但现在这个 bug 崩溃发生次数蹭蹭涨，崩溃占比高了起来，不得不重视起来。

点进崩溃详情，只有短短的一句 `Can't add self as subview`。

<!--more-->

崩溃调用堆栈解析出来如下：

![崩溃调用堆栈解析](http://7xjhyk.com1.z0.glb.clouddn.com/QQ20151031-0@2x.png)

对解决问题根本起不了作用，只能依靠 Google 了。

## bug 复现

在 StackOverflow 上找到复现此 bug 的方法，尝试同时 `push` 两个控制器，或者同时 `push` 和 `pop` 一个控制器。

![测试同时 push 两个控制器](http://i.stack.imgur.com/93n6V.png)

上图是当时一回答此问题人的运行结果，但现在测试并不会引起崩溃，不过同时 `push` 一个控制器是会导致崩溃的，测试结果如下图：

![测试同时 push 一个控制器](http://7xjhyk.com1.z0.glb.clouddn.com/QQ20151105-0@2x.png)

## 解决办法

创建一个分类，拦截控制器入栈\出栈的方法调用，通过安全的方式，确保当有控制器正在进行入栈\出栈操作时，没有其他入栈\出栈操作。

此分类用到运行时 (Runtime) 的方法交换 `Method Swizzling`，因此只需要复制下面的代码到自己的项目中，此 bug 就不复存在了。

```objc
#import "UINavigationController+Consistent.h"
#import <objc/runtime.h>
/// This char is used to add storage for the isPushingViewController property.
static char const * const ObjectTagKey = "ObjectTag";

@interface UINavigationController ()
@property (readwrite,getter = isViewTransitionInProgress) BOOL viewTransitionInProgress;
@end

@implementation UINavigationController (Consistent)
- (void)setViewTransitionInProgress:(BOOL)property {
    NSNumber *number = [NSNumber numberWithBool:property];
    objc_setAssociatedObject(self, ObjectTagKey, number , OBJC_ASSOCIATION_RETAIN);
}

- (BOOL)isViewTransitionInProgress {
    NSNumber *number = objc_getAssociatedObject(self, ObjectTagKey);
    return [number boolValue];
}

#pragma mark - Intercept Pop, Push, PopToRootVC
/// @name Intercept Pop, Push, PopToRootVC
- (NSArray *)safePopToRootViewControllerAnimated:(BOOL)animated {
    if (self.viewTransitionInProgress) return nil;
    if (animated) {
        self.viewTransitionInProgress = YES;
    }
    //-- This is not a recursion, due to method swizzling the call below calls the original  method.
    return [self safePopToRootViewControllerAnimated:animated];
}

- (NSArray *)safePopToViewController:(UIViewController *)viewController animated:(BOOL)animated {
    if (self.viewTransitionInProgress) return nil;
    if (animated) {
        self.viewTransitionInProgress = YES;
    }
    //-- This is not a recursion, due to method swizzling the call below calls the original  method.
    return [self safePopToViewController:viewController animated:animated];
}

- (UIViewController *)safePopViewControllerAnimated:(BOOL)animated {
    if (self.viewTransitionInProgress) return nil;
    if (animated) {
        self.viewTransitionInProgress = YES;
    }
    //-- This is not a recursion, due to method swizzling the call below calls the original  method.
    return [self safePopViewControllerAnimated:animated];
}

- (void)safePushViewController:(UIViewController *)viewController animated:(BOOL)animated {
    self.delegate = self;
    //-- If we are already pushing a view controller, we dont push another one.
    if (self.isViewTransitionInProgress == NO) {
        //-- This is not a recursion, due to method swizzling the call below calls the original  method.
        [self safePushViewController:viewController animated:animated];
        if (animated) {
            self.viewTransitionInProgress = YES;
        }
    }
}

// This is confirmed to be App Store safe.
// If you feel uncomfortable to use Private API, you could also use the delegate method navigationController:didShowViewController:animated:.
- (void)safeDidShowViewController:(UIViewController *)viewController animated:(BOOL)animated {
    //-- This is not a recursion. Due to method swizzling this is calling the original method.
    [self safeDidShowViewController:viewController animated:animated];
    self.viewTransitionInProgress = NO;
}


// If the user doesnt complete the swipe-to-go-back gesture, we need to intercept it and set the flag to NO again.
- (void)navigationController:(UINavigationController *)navigationController willShowViewController:(UIViewController *)viewController animated:(BOOL)animated {
    id<UIViewControllerTransitionCoordinator> tc = navigationController.topViewController.transitionCoordinator;
    [tc notifyWhenInteractionEndsUsingBlock:^(id<UIViewControllerTransitionCoordinatorContext> context) {
        self.viewTransitionInProgress = NO;
        //--Reenable swipe back gesture.
        self.interactivePopGestureRecognizer.delegate = (id<UIGestureRecognizerDelegate>)viewController;
        [self.interactivePopGestureRecognizer setEnabled:YES];
    }];
    //-- Method swizzling wont work in the case of a delegate so:
    //-- forward this method to the original delegate if there is one different than ourselves.
    if (navigationController.delegate != self) {
        [navigationController.delegate navigationController:navigationController
                                     willShowViewController:viewController
                                                   animated:animated];
    }
}

+ (void)load {
    //-- Exchange the original implementation with our custom one.
    method_exchangeImplementations(class_getInstanceMethod(self, @selector(pushViewController:animated:)), class_getInstanceMethod(self, @selector(safePushViewController:animated:)));
    method_exchangeImplementations(class_getInstanceMethod(self, @selector(didShowViewController:animated:)), class_getInstanceMethod(self, @selector(safeDidShowViewController:animated:)));
    method_exchangeImplementations(class_getInstanceMethod(self, @selector(popViewControllerAnimated:)), class_getInstanceMethod(self, @selector(safePopViewControllerAnimated:)));
    method_exchangeImplementations(class_getInstanceMethod(self, @selector(popToRootViewControllerAnimated:)), class_getInstanceMethod(self, @selector(safePopToRootViewControllerAnimated:)));
    method_exchangeImplementations(class_getInstanceMethod(self, @selector(popToViewController:animated:)), class_getInstanceMethod(self, @selector(safePopToViewController:animated:)));
}
@end
```



