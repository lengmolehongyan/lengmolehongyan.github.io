---
layout: post
title: "UITableView tableHeaderView, tableFooterView 子控件布局约束警告"
date: 2016-03-26 09:59:58 +0800
comments: true
categories: ios开发小细节
keywords: UITableView,tableHeaderView,tableFooterView,自动布局,自动布局冲突
description: UITableView的tableHeaderView,tableFooterView添加子控件自动布局约束冲突
---

带有 `tableHeaderView`，`tableFooterView` 的 `UITableView` 在实际项目中很常见，添加了头部视图或者底部视图，会嵌在 `UITableView` 的头部或者底部，跟随 `UITableView` 一起滚动。

有关 `tableHeaderView`，`tableFooterView` 介绍，文档和头文件已经说明的很清楚。

```objc
// 显示在表视图上方的附属视图，默认为 nil，不同于组头视图
@property (nonatomic, strong, nullable) UIView *tableHeaderView;
// 显示在表视图下方的附属视图，默认为 nil，不同于组尾视图
@property (nonatomic, strong, nullable) UIView *tableFooterView;
```

<!--more-->

![Demo 界面]({{root_url}}/images/QQ20160410-0@2x.png)

如上界面，在 `UITableView` 顶部有个 `tableHeaderView`，里面有一个 `UILabel`，label 的文字可能过长需要换行。使用自动布局布局 `tableHeaderView` 内部这个 label 子控件，我们可能想当然的把约束按照如下这样写：

```objc
... // 创建 tableView

CGFloat screenWidth = [UIScreen mainScreen].bounds.size.width;
CGFloat tableHeaderHeight = 100.0;
UIView *tableHeader = [[UIView alloc] initWithFrame:CGRectMake(0, 0, screenWidth, tableHeaderHeight)];
tableHeader.backgroundColor = [UIColor lightGrayColor];
tableView.tableHeaderView = tableHeader;

UILabel *headerLabel = [[UILabel alloc] init];
headerLabel.translatesAutoresizingMaskIntoConstraints = NO;
headerLabel.numberOfLines = 0;
headerLabel.text = @"这是 tableView 的 tableHeaderView 中的一个 label";
headerLabel.textColor = [UIColor greenColor];
[tableHeader addSubview:headerLabel];

CGFloat leftMargin = 15.0;
CGFloat labelWidth = screenWidth - 2*leftMargin;
NSLayoutConstraint *labelConstraint1 = [NSLayoutConstraint constraintWithItem:headerLabel attribute:NSLayoutAttributeLeft relatedBy:NSLayoutRelationEqual toItem:tableHeader attribute:NSLayoutAttributeLeft multiplier:1.0 constant:leftMargin];
NSLayoutConstraint *labelConstraint2 = [NSLayoutConstraint constraintWithItem:headerLabel attribute:NSLayoutAttributeCenterY relatedBy:NSLayoutRelationEqual toItem:tableHeader attribute:NSLayoutAttributeCenterY multiplier:1.0 constant:0];
NSLayoutConstraint *labelConstraint3 = [NSLayoutConstraint constraintWithItem:headerLabel attribute:NSLayoutAttributeRight relatedBy:NSLayoutRelationEqual toItem:tableHeader attribute:NSLayoutAttributeRight multiplier:1.0 constant:-leftMargin];
[tableHeader addConstraints:@[labelConstraint1, labelConstraint2, labelConstraint3]];
```

这样布局出来的 label，显示上是没有问题的，也的确会换行，但是 console 却会报 `Unable to simultaneously satisfy constraints` 警告：

![Unable to simultaneously satisfy constraints]({{root_url}}/images/QQ20160424-0@2x.png)

向一个普通的 `UIView` 中添加一个多行的 `UILabel` 子控件，我的约束布局习惯是在 `x` 方向上只需要指定距离父控件 view 的左右间距，再指定它的 `y` 方向上的位置，一般情况下，这个 label 的布局已经是完成了，而且没有问题。可是，在这里的 `tableHeaderView` 中的 label 布局为什么出问题呢？

原因在于 `UITableView` 对 `tableHeaderView` 属性的引用关系，`tableHeaderView` 默认是没有的，给 `UITableView` 添加 `tableHeaderView` 时，需要创建一个 view 然后赋值给 `UITableView` 的 `tableHeaderView` 属性。

创建一个 `tableHeaderView` 的 view 时，一般使用 `- (instancetype)initWithFrame:(CGRect)frame;` 方法，这里需要传入 view 的 `frame` 参数，`frame` 参数可能会这样传入 `CGRectMake(0, 0, screenWidth, tableHeaderHeight)`，误以为需要将 view 的宽度设置一下，而问题恰巧就出现在这里的宽度。

当代码走过创建 view 那一行，从 log 中看到，view 的 `frame` 确实和我们设置的一样：

![创建 view 后的 frame]({{root_url}}/images/QQ20160424-1@2x.png)

但是，当调用 `tableView.tableHeaderView = tableHeader;` 后，会发现此时 view 的 `frame` 的宽度为 0 了：

![指定 view 为 tableHeaderView 后的 frame]({{root_url}}/images/QQ20160424-2@2x.png)

这也就是为什么 console 会报出约束警告的原因，对于一个宽度为 0 的 view，而内部的 label 子控件 `x` 方向上的约束却以其父控件 view 的左右间距来布局，不报约束警告才怪呢？！

想要解决这个约束警告很简单，只需指定 `x` 方向左边离父控件 view 的左边距离 15 个点，`y` 方向上按需要设定，对于有固有内容尺寸的控件，设定这两个约束就可以了，但是由于这是个需要换行的 label，因此还需要限制 label 的宽度，当然也可以指定 label 的 `preferredMaxLayoutWidth` 的值。

```objc
CGFloat leftMargin = 15.0;
CGFloat labelWidth = screenWidth - 2*leftMargin;

// 解决约束警告方式一:
NSLayoutConstraint *labelConstraint1 = [NSLayoutConstraint constraintWithItem:headerLabel attribute:NSLayoutAttributeLeft relatedBy:NSLayoutRelationEqual toItem:tableHeader attribute:NSLayoutAttributeLeft multiplier:1.0 constant:leftMargin];
NSLayoutConstraint *labelConstraint2 = [NSLayoutConstraint constraintWithItem:headerLabel attribute:NSLayoutAttributeCenterY relatedBy:NSLayoutRelationEqual toItem:tableHeader attribute:NSLayoutAttributeCenterY multiplier:1.0 constant:0];
NSLayoutConstraint *labelWidthConstraint = [NSLayoutConstraint constraintWithItem:headerLabel attribute:NSLayoutAttributeWidth relatedBy:NSLayoutRelationEqual toItem:nil attribute:0 multiplier:1 constant:labelWidth];
[tableHeader addConstraints:@[labelConstraint1, labelConstraint2]];
headerLabel.preferredMaxLayoutWidth = labelWidth;

// 解决约束警告方式二:
NSLayoutConstraint *labelConstraint1 = [NSLayoutConstraint constraintWithItem:headerLabel attribute:NSLayoutAttributeLeft relatedBy:NSLayoutRelationEqual toItem:tableHeader attribute:NSLayoutAttributeLeft multiplier:1.0 constant:leftMargin];
NSLayoutConstraint *labelConstraint2 = [NSLayoutConstraint constraintWithItem:headerLabel attribute:NSLayoutAttributeCenterY relatedBy:NSLayoutRelationEqual toItem:tableHeader attribute:NSLayoutAttributeCenterY multiplier:1.0 constant:0];
NSLayoutConstraint *labelConstraint3 = [NSLayoutConstraint constraintWithItem:headerLabel attribute:NSLayoutAttributeRight relatedBy:NSLayoutRelationEqual toItem:tableHeader attribute:NSLayoutAttributeRight multiplier:1.0 constant:-leftMargin];
NSLayoutConstraint *labelWidthConstraint = [NSLayoutConstraint constraintWithItem:headerLabel attribute:NSLayoutAttributeWidth relatedBy:NSLayoutRelationEqual toItem:nil attribute:0 multiplier:1 constant:labelWidth];
[tableHeader addConstraints:@[labelConstraint1, labelConstraint2]];
[headerLabel addConstraint:labelWidthConstraint];
```

