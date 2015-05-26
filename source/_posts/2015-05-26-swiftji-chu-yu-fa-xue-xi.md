---
layout: post
title: "swift基础语法学习"
date: 2015-05-26 22:12:42 +0800
comments: true
categories: "iOS开发"
---

### 变量和常量

* `var`**变量**，可以修改的
* `let`**常量**，一经定义不能修改
* 在swift开发中，通常先定义常量`let`，只有必须修改的时候，再改成`var`

```swift
// 实例化一个UIView对象 保存在堆中
let view = UIView(frame: CGRectMake(0, 0, 100, 100))
// 修改的是view的属性 并没有修改view本身的地址
view.backgroudColor = UIColor.redColor()
```

* 注意点:
	* swift是一个对变量类型要求及其严格的语言
	* 任何数据类型之间，都不能隐式转换，如果要在不同类型之间进行计算，必须转换格式
	* 整数的格式是`Int`对应于OC中的`long`(64位的)，小数的格式是`Double`，OC中默认的小数格式`CGFloat`
	* 数据类型的推导是在给变量设置初始值的时候，根据“右边”来判断的
	* 在真正初始化的时候，才能决定变量的准确类型
* 在定义变量的时候，可以直接指定变量的类型，便于后续的计算不需要转换

```swift
let a: Double = 10
let b = 10.5
let c = a + b
```

***

### 分支

* `if`在C语言中有一个特点:非零即真，在swift中，没有非零即真的概念，只有`true/false`，在编写分支语句时，必须准确的指定条件的真假。
* 格式:
	* 条件不需要括号
	* 必须要有`{}`
* `convenience init?()`表示一个函数未必能够真的实例化出来一个对象，在swift中，要求在编写代码的时候，必须考虑这些问题，能够尽早地发现问题

```swift
// 使用if同时设置数值
if let url = NSURL(string: "http://www.baidu.com/s?word=he") {
    // 代码执行到此 url就一定有值 所以不需要再使用!
    let request = NSURLRequest(URL: url)
}
```

* 实际的应用技巧:
	* `?`表示可以有值，也可以没有值
	* 打印可选项的时候，同时会输出一个`Optional`，提示开发者，这是一个可选项，苹果把判断对象是否有内容的工作交给了程序员
	* `??`用来快速判断对象是否为`nil`

```swift
var str: NSString
str = "hello"
let a = 10
// 存在的风险 如果str没有设置初始值 会直接崩溃
// let len = a + str!.length
// ?? 用来快速判断对象是否为nil
let len2 = a + (str?.length ?? 0)
```

***

### 循环

* 传统的写法，和C语言几乎一样，需要注意的是，需要使用`var`而不是`let`

```swift
for var i = 0; i < 10; i++ {
    println(i)
}
```

* 方便写法，`in`用来指定范围

```swift
// 范围0~9
for i in 0..<10 {
    println(i)
}
// 范围0~10
for i in 0...10 {
    println(i)
}
```

* 如果遍历的时候，对索引下标不关注，在swift中，`_`使用的非常广泛，主要用于忽略

```swift
for _ in 0...5 {
    println("hello")
}
```

***

### 字符串

* 在swift中，**字符串**默认的类型是`String`，而不是`NSString`
* swift中`String`是一个结构体，执行效率更高，支持快速遍历。`NSString`继承自`NSObject`，是一个OC对象，不支持快速遍历

```swift
let str = "Hello!"
// 对字符串的快速遍历
for c in str {
	println(c)
}
```

* 字符串的拼接

```swift
let str1 = "Hello"
let str2 = str1 + "World!"
```

* 格式字符串

```swift
let str = String(format: "%02d:%02d:%02d", arguments: [1, 2, 3])
```

* 在swift中，如果要结合`range`一起使用字符串，建议先转成`NSString`再处理

```swift
let str: NSString = "Hello, world!"
let subS = str.substringWithRange(NSMakeRange(1, 3))
```

***

### 数组

* 使用`[]`定义**数组**，数组类型由中括号里面元素决定
* OC中数组分为可变数组和不可变数组，swift中`let`是不可变的，`var`是可变的，不能向不可变数组中追加元素
* 数组遍历:

```swift
// 数组遍历
let array = ["name", "age", "no"]
for str in array {
	println(str)
}
```

* 可变数组添加元素

```swift
var arrayM = ["name", "age"]
arrayM.append("no")
```

* 如果定义数组时，指定的对象类型不一致，定义的数组类型是`[NSObject]`。OC中，如果要向数组中添加数字，需要转换成`NSNumber`，swift中，可以直接添加
* 数组的定义和初始化

```swift
// 定义但是没有初始化
var arrayM: [String]
// 初始化一个字符串的可变数组
arrayM = [String]()
arrayM.append("name")
```

***

### 字典

* 定义一个**字典**，仍然使用`[]`，在目前的swift版本中，定义字典通常使用`[String: NSObject]`，大多数情况下，`key`的类型是固定的
* `let`是不可变的，`var`是可变的
* 如果`key`已经存在，利用这个`key`设置数据时，会覆盖之前的值
* 字典的特性:`key`是不允许重复的
* 字典的遍历:

```swift
// 定义并且实例化字典
var dict1 = [String: NSObject]()
dict1["name"] = "zhangsan"
dict1["age"] = 20
// 注意:k, v可以随便写，但是，前面是key，后面是value
for (kk, vv) in dict1 {
    println("\(kk), \(vv)")
}
```

***

### 函数

* **函数**的定义格式:
	* `func 函数名(参数列表) -> 返回值 {// 代码实现}`
	* `->`是swift特有的，表示前面的执行结果，输出给后面的
	* 强制填写参数，使用`#`，可以在调用的时候，会让代码提示更直观
* 如果没有返回值`-> 返回值`可以省略，下面三种方式等价
	* `-> Void`
	* `-> ()`
	* 完全忽略

```swift
// 定义函数
func sum(#a: Int, #b: Int) -> Int {
    return a + b
}
// 函数调用
sum(a: 10, b: 20)
```

***

### 闭包

* **闭包**和OC中的`block`类似，但是OC中的`block`是一个匿名函数，swift中**函数**是**闭包**的一个特例
* 闭包的使用，几乎和OC中的`block`是一样的
	* 提前准备好代码
	* 在需要的时候执行
* 闭包的格式:
	* `{ (参数) -> () in // 准备执行的代码实现 }`
	* `in`是定义和代码实现之间的分隔
* 闭包的简写:
	* 如果闭包是最后一个参数，可以进行简化
	* 称为“尾随闭包”

```swift
override func viewDidLoad() {
        super.viewDidLoad()
        demo(10, completion: {(str) -> () in
            println("回调后\(str)")
        })
    }
    
func demo(a: Int, completion:(str: String) -> ()) {
        println("回调前\(a)")
        // 执行回调
        completion(str: "\(a)")
}
// 输出回调前10 回调后10
```

* 闭包的**返回值**

```swift
override func viewDidLoad() {
   super.viewDidLoad()
   demo { () -> Int in
       return 20
   }
   // 输出行高20
}
    
func demo(rowH:() -> Int) {
   let rowH = rowH()
   println("行高\(rowH)")
}
```

* 定义**闭包属性**的时候，有两种选择
	* 直接设置初始值
	* 设置一个可选项`?`

```swift
// 定义方式 表示闭包可以为空
var completion: (()->())?
```

* 闭包的**循环引用**
	* `block`中出现`self`的时候，为了保证代码安全，在定义闭包的时候，需要对外部变量进行拷贝
	* OC中解决循环引用的办法`__weak typeof(self) weakSelf = self`
	* 在swift中，如果直接访问属性数值，可以省略`self.`
	* swift中默认都是强引用，如果需要弱引用，可以使用`weak`修饰符号
	* 在闭包中，必须要明确的使用`self`，因为闭包是提前准备的代码，不知道什么时候会执行
	* 在OC中判断是否有循环引用，使用`-dealloc`方法，在swift中`deinit`叫做**析构函数**，就是在对象被释放前执行，与OC中的`-dealloc`方法类似

```swift
weak var weakSelf = self
demo4 {
	// 使用?的好处 就是一旦 self 被释放，就什么也不做
	weakSelf?.view.backgroundColor = UIColor.redColor()
}
```

