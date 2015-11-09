---
layout: post
title: "Block 的简单使用"
date: 2015-09-10 23:42:34 +0800
comments: true
categories: 
---

### Block 的介绍

* 对象与对象之间的通信方式
	1. **代理-协议**，**通知**，**Block**。
	2. 三种通信方式都实现了对象之间的解耦合。
	3. 通知的通信方式是 **1对多**。
	4. 代理、Block 是 **1对1**。

<p></p>

* Block 介绍
	1. Block 是 iOS4.0 之后新增的一种语法结构，也称为“闭包”。
	2. Block 是一个匿名的函数代码块，此代码块可以作为参数传递给其他对象。
	3. 可以把 block 当做 Objective-C 的匿名函数，block 是 OC 中的一种数据类型，`^`是 block 的特有标记。

<p></p>

* Block 格式说明
	1. `(返回类型)(^block名称)(参数类型)=^(参数列表){代码实现};`。
	2. 如果没有参数，等号后面参数列表的()可以省略。

<p></p>

<!--more-->

### Block 的使用

* Block 的定义

![定义Block的常见两种方式](http://i2.tietuku.com/4a270839c9929017.png)

* Block 的3种类型
	1. 不管在 ARC 还是 MRC 环境下，block 内部如果没有访问外部变量，这个 block 是 **全局 block** `__NSGlobalBlock__`，形式类似函数，存储在内存中的 **代码区**。
	2. 在 **MRC** 下，block 内部如果访问外部变量，这个 block 是 **栈 block** `__NSStackBlock__`，存储在内存中的 **栈** 上。
	3. 在 **MRC** 下，block 内部访问外部变量，同时对该 block 做一次 **copy 操作**，这个 block 是 **堆 block** `__NSMallocBlock__`，存储在内存中的 **堆** 上。
	4. 在 **ARC** 下，block 内部如果访问外部变量，这个 block 是 **堆 block** `__NSMallocBlock__`，存储在内存中的 **堆** 上，因为在 ARC 下，默认对 block 做了一次 copy 操作。

![全局block](http://i2.tietuku.com/8c06ba2fe062559a.png)
![栈block](http://i2.tietuku.com/bb203edf923cea6b.png)
![堆block](http://i2.tietuku.com/5c461733e51097cd.png)

* Block 作为方法的参数
	* 将 block 作为方法的参数，可以用 block 来封装代码块。

![block作为方法的参数](http://ww1.sinaimg.cn/large/9491566bjw1erczgvtdkdj211i0d2aep.jpg)

* Block 作为属性

![block作为属性](http://ww4.sinaimg.cn/large/9491566bjw1ercziuw0btj21420hin58.jpg)

* Block 的方式遍历数组\字典

![block的方式遍历数组\字典](http://ww1.sinaimg.cn/large/9491566bjw1erd0j2s7stj213s0l2qfl.jpg)


### Block 的使用注意

* Block 访问外部变量
	1. block 内部可以访问外部的变量，block 默认是将其复制到其数据结构中来实现访问的。
	2. 默认情况下，block 内部不能修改外面的局部变量，因为通过 block 进行闭包的变量是 const 的。
	3. 给局部变量加上 `__block` 关键字，这个局部变量就可以在 block 内部修改，block 是复制其引用地址来实现访问的。

![block访问外部变量](http://ww2.sinaimg.cn/large/9491566bjw1erdasuqz67j216s0kqtl6.jpg)

* Block 作为属性应该用 copy 修饰
	1. 当用 weak、assign 修饰 block 属性时，block 访问外部变量，此时 block 的类型是 **栈 block**。保存在栈中的 block，当 block 所在函数\方法返回\结束，该 block 就会被销毁。在其他方法内部调用访问该 block，就会引发野指针错误 `EXC_BAD_ACCESS`。
	2. 当用 copy、strong 修饰 block 属性时，block 访问外部变量，此时 block 的类型是 **堆 block**。保存在堆中的 block，当引用计数器为0时被销毁，该类型 block 是由栈类型的 block 从栈中复制到堆中形成的，因此可以在其他方法内部调用该 block。在 ARC 下，`strong` 和 `copy` 都可以用来修饰 block，但是建议修饰 block 属性使用 `copy`。

![weak修饰block](http://ww1.sinaimg.cn/large/9491566bjw1erdce4hllbj21kw0j2qc7.jpg)

![copy修饰block](http://ww4.sinaimg.cn/large/9491566bjw1erdfvoacrfj214i0jmqbu.jpg)


推荐阅读：[Objective-C中的Block](http://www.cocoachina.com/ios/20150109/10891.html)


