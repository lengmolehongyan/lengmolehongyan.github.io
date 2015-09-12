---
layout: post
title: "KVC 的简单使用"
date: 2015-09-12 22:58:12 +0800
comments: true
categories: 
---

### KVC 字典转模型

* KVC 中经常使用的就是字典转模型

```objc
// NSObject(NSKeyValueCoding) NSObject 的分类
- (void)setValuesForKeysWithDictionary:(NSDictionary *)keyedValues;
```

![字典转模型](http://i1.tietuku.com/cc3c54dccb414089.png)

***

<!--more-->

### KVC 的大招

* KVC 设置对象属性及取值

```objc
- (void)setValue:(id)value forKey:(NSString *)key;
- (id)valueForKey:(NSString *)key;
```

![LNPerson类的头文件](http://i2.tietuku.com/4d55944da6a1bcf7.png)
![KVC设置对象属性及取值](http://i1.tietuku.com/12c24aeb2ed198ac.png)

* KVC 间接设置对象属性
	* 在运行的时候，KVC 可以间接设置对象的属性，不管对象属性是否在 `.h` 中公开，当然这违背面向对象设计的 **开闭原则**，严重不建议在程序开发中使用。

![LNPerson类的.m文件](http://i1.tietuku.com/ec730d8853185ebc.png)
![KVC间接设置对象属性](http://i1.tietuku.com/b5dba559d850fe72.png)

***

### KVC 模型转字典

* KVC 模型转字典
	* KVC 模型转字典，参数是属性名称的数组。

```objc
// keys 是属性名称的数组
- (NSDictionary *)dictionaryWithValuesForKeys:(NSArray *)keys;
```

![KVC模型转字典](http://i2.tietuku.com/8b2fb0382d0bce99.png)

***

### KVC 核心动画

* KVC 最经典的应用——核心动画
	* 通过 KVC 设置动画的 `KeyPath`，在实例化动画的时候，指定图层的可动画属性。

![](http://i1.tietuku.com/2df35a7c17406ce3.png)
![核心动画](http://i1.tietuku.com/1545fe7a18db2738.gif)

***


