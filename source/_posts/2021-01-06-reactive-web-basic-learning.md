---
layout: post
title: "响应式 Web 基础学习"
date: 2021-01-06 14:56:44 +0800
comments: true
categories: 
keywords: Web,响应式,Reactive,learning,H5,HTML,CSS,HTML5
description: 响应式 Web 基础学习,reactive web basic learning
---

本文为学习 [FreeCodeCamp 网站](https://learn.freecodecamp.one/) 《响应式 Web 设计》章节的总结

## HTML 基础

* HTML 指的是超文本标记语言，全称：HyperText Markup Language
* HTML 不是一种编程语言，而是一种标记语言
* HTML 使用标记标签来描述网页
* HTML 文档包含了 HTML 标签及文本内容
* `<!DOCTYPE ...>` 声明当前页面使用的 HTML 的版本信息，`...` 部分就是版本的数字，`<!DOCTYPE html>` 对应的就是 HTML5

### 元素

一个 HTML 元素包含了开始标签（起始标签 opening tag）与结束标签（闭合标签 closing tag），切记所有标签都需要**闭合**（保证兼容性），标签写法要用**小写**字母

常见元素：

* `<html></html>` 整个 HTML 文档，HTML 页面的根元素
* `<head></head>` HTML 文档元数据，`link`、`meta`、`title`、`style` 元素都应该放入 `head` 中
* `<body></body>` HTML 文档的主体
* `<h1></h1>` 主标题
* `<h2></h2>` 副标题
* `<h3></h3>` ... `<h6></h6>` 不同级别的标题
* `<div></div>` 常用于组合块级元素，盛装其他元素的通用容器
* `<p></p>` 段落
* `<br />` 换行
* `<a href="xxx" />` 锚点、链接
* `<img src="xxx" />` 图像

> `href` 是 Hypertext Reference 的缩写，表示超文本引用，用来建立当前元素和文档之间的链接，常用的有：`link`、`a` 元素。例如：`<link href="reset.css" rel="stylesheet"/>`，浏览器会识别该文档为 CSS 文档，并行下载该文档，并且不会停止对当前文档的处理，这也是建议使用 `link`，而不采用 `@import` 加载 CSS 的原因，详情可查看[《CSS 引入方式》](https://segmentfault.com/a/1190000003866058) 文章
>
> `src` 是 source 的缩写，`src` 的内容是页面必不可少的一部分，是引入，`src` 指向的内容会嵌入到文档中当前标签所在的位置，常用的有：`img`、`script`、`iframe` 元素。例如：`<script src="script.js"></script>`，当浏览器解析到该元素时，会暂停浏览器的渲染，直到该资源加载完毕，这也是将 JS 脚本放在底部而不是头部得原因
> 
> `src` 用于替换当前元素，`href` 用于在当前文档和引用资源之间建立联系

#### 图像 `img` 元素

```html
<img src="xxx" alt="xxx" width="xx" height="xx" />
```

* 建议所有的 `img` 元素必须有 `alt` 属性，`alt` 属性的文本是当图片无法加载时显示的替代文本
* 如果 `img` 元素设置了高度宽度，页面加载时会保留指定的尺寸，反之没有设置，加载页面时有可能会破坏 HTML 页面的整体布局

#### 链接、锚点 `a` 元素

```html
<a href="xxx" target="_blank">链接文本</a>
```

* 链接文本不必一定是文本，图片或其他 HTML 元素都可以成为链接
* 设置 `target="_blank"` 点击链接会打开新的标签页
* 设置锚点的 `href` 属性值为井号 `#` 加上跳转区域对应的 `id` 属性值，可以用来网页内不同区域的跳转
* `id` 是用来描述网页元素的一个属性，它的值在整个页面中唯一
* `href="#"` 代表跳转的链接为固定链接

#### 列表 `ol`、`ul` 元素

```html
<!--无序列表-->
<ul>
    <li>xxx</li>
</ul>
<!--有序列表-->
<ol>
    <li>xxx</li>
</ol>
```

* `li` 元素定义列表项
* 列表项内部可以使用段落、换行符、图片、链接以及其他列表等

#### 表单 `form` 元素

```html
<form>
    <input type="xxx" name="xxxx" />
</form>
```

* 输入类型由类型属性 `type` 决定，常用类型：
  * 文本输入框 `type="text"` 
  * 密码输入框 `type="password"`
  * 单选按钮 `type="radio"`
  * 多选按钮 `type="checkbox"`
  * 提交按钮 `type="submit"`
  * 重置按钮 `type="reset`
* 表单元素本身并不可见
* `input` 元素的 `required` 属性代表该元素是否为必填字段
* `input` 元素的 `checked` 属性代表该元素中的单选、多选按钮是否被默认选中
* 所有关联的（同一组）单选按钮应拥有相同的 `name` 属性

## CSS 基础

![CSS 语法](https://www.runoob.com/wp-content/uploads/2013/07/632877C9-2462-41D6-BD0E-F7317E4C42AC.jpg)

* CSS 指的是层叠样式表，全称：Cascading Style Sheets
* 样式定义如何显示 HTML 元素，样式通常存储在样式表中
* 外部样式表通常存储在 CSS 文件中，外部样式表可以极大提高工作效率
* CSS 由两个主要的部分构成：选择器，以及一条或多条声明
* CSS 声明总是以分号 `;` 结束，总是以大括号 `{}` 括起来

### 样式

#### 外部样式表 External style sheet

使用外部样式表可以通过改变一个文件来改变整个站点的外观，每个页面使用 `link` 元素链接到具体的样式表文件

```html
<head>
    <link rel="stylesheet" type="text/css" href="xxxx.css">
</head>
```

浏览器会从文件 `xxxx.css` 中读到样式声明，并根据它来格式化文档

#### 内部样式表 Internal style sheet

当单个文档需要特殊的样式时，应该使用内部样式表

```html
<head>
    <style>
        p {margin-left: 10px;}
    </style>
</head>
```

#### 内联样式 Inline style

```html
<p style="margin-left: 10px;">xxxx</p>
```

#### 多重样式优先级

内联样式 > 内部样式表 > 外部样式表 > 浏览器默认样式

### 选择器

#### id 选择器

为标有特定 id 的 HTML 元素指定特定的样式，使用 `#` 标识 selector，语法格式：

* `#S{...}`（S 为元素对应的 id 属性值）

#### class 选择器

class 选择器可在多个元素中使用，使用 `.` 标识 selector，语法格式：

* `.S{...}`（S 为元素对应的类名）

#### 元素选择器

使用标签名作为 selector 名，语法格式：

* `S{...}`（S 为元素名）

#### 包含选择器

指定目标选择器必须处在某个选择器对应的元素内部，语法格式：

* `A B{...}`（A、B 为 HTML 元素名，表示对处于 A 元素中的 B 元素有效）
* `.A B{...}`（A 为元素对应的类名，B 为 HTML 元素名，表示对处于类名为 A 的元素中的 B 元素有效）

#### 子选择器

类似于包含选择器，区别在于，子选择器只允许直接子代元素匹配相应的样式，强制指定目标选择器作为包含选择器对应的元素内部的元素，语法格式：

* `A>B{...}`（A、B 为 HTML 元素名，表示对处于 A 元素中的直接子代 B 元素有效）
* `.A>B{...}`（A 为元素对应的类名，B 为 HTML 元素名，表示对处于类名为 A 的元素中的直接子代 B 元素有效）

#### 兄弟选择器

语法格式：

* `A~B{...}`（A、B 为 HTML 元素名，表示对和 A 元素处于同级的 B 元素有效）

#### 相邻选择器

语法格式：

* `A+B{...}`（A、B 为 HTML 元素名，表示对和 A 元素处于同级的且严格相邻的 B 元素有效）

#### 通用选择器

匹配 HTML 中的所有元素，语法格式：

* `*{...}`

更详细的 CSS 样式优先级介绍，可去 [CSS 样式优先级](https://www.runoob.com/w3cnote/css-style-priority.html) 该篇文章查看

### CSS 盒模型

“盒模型” 这一术语是用来设计和布局时使用，CSS 盒模型本质上是一个盒子，封装周围的 HTML 元素，盒模型允许在其它元素和周围元素边框之间的空间放置元素

![CSS 盒模型](https://www.runoob.com/images/box-model.gif)

#### 块级盒子（Block Box）

表现行为：

* 盒子会在内联的方向上扩展并占据父容器在该方向上的所有可用空间，在绝大数情况下意味着盒子会和父容器一样宽
* 每个盒子都会换行
* `width` 和 `height` 属性可以发挥作用
* `padding`, `margin` 和 `border` 会将其他元素从当前盒子周围推开

标题（`<h1></h1>` 等）和段落（`<p></p>`）默认情况下都是块级盒子

#### 内联盒子（Inline Box）

表现行为：

* 盒子不会产生换行
* `width` 和 `height` 属性将不起作用
* 垂直方向的 `padding`、`margin` 以及 `border` 会被应用但是不会把其他处于 `inline` 状态的盒子推开
* 水平方向的 `padding`、`margin` 以及 `border` 会被应用且会把其他处于 inline 状态的盒子推开

`<a></a>`、`<span></span>`、`<em></em>` 以及 `<strong></strong>` 默认情况下都是内联盒子

#### CSS3 弹性盒子（Flex Box）

CSS3 弹性盒子是一种当页面需要适应不同的屏幕大小以及设备类型时确保元素拥有恰当的行为的布局方式。弹性盒子由弹性容器（Flex container）和弹性子元素（Flex item）组成，弹性容器通过设置 `display` 属性的值为 `flex` 或 `inline-flex` 将其定义为弹性容器

![CSS3 弹性盒子模型](https://developer.mozilla.org/files/3739/flex_terms.png)

* 主轴（main axis）：沿着 flex 元素放置的方向延伸的轴，该轴的开始和结束被称为 main start 和 main end
* 交叉轴（cross axis）：垂直于 flex 元素放置方向的轴，该轴的开始和结束被称为 cross start 和 cross end
* 设置了 `display: flex` 的父元素被称之为 flex 容器（flex container）
* 在 flex 容器中表现为柔性的盒子的元素被称之为 flex 项（flex item）
