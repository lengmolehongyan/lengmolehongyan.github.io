---
layout: post
title: "react-native-route-flux 使用文档"
date: 2018-03-21 11:11:40 +0800
comments: true
categories: 
keywords: react-native-route-flux
description: react-native-route-flux 使用文档
---

## 迷你教程

![](https://github.com/aksonov/react-native-router-flux/blob/v3/docs/super_simple.gif?raw=true)

在这个简单例子中，只需要三个文件：

1. 根 index 文件：`index.js`
2. 自动加载的第一个页面：`PageOne.js`
3. 需要跳转到的第二个页面：`PageTwo.js`

### index.js

```js
import React, { Component } from 'react';
import { Router, Scene } from 'react-native-router-flux';

import PageOne from './PageOne';
import PageTwo from './PageTwo';

export default class App extends Component {
  render() {
    return (
      <Router>
        <Scene key="root">
          <Scene key="pageOne" component={PageOne} title="PageOne" initial={true} />
          <Scene key="pageTwo" component={PageTwo} title="PageTwo" />
        </Scene>
      </Router>
    )
  }
}
```

在 `react-native-router-flux` 中，路由（或者页面）被称为 `<Scene>`。通常情况下，你的 Scenes 应该被封装在根  Scene 中，然后才能包装在 `render()` 函数返回的 `<Router>` 组件中。

任何一个 `<Scene>` 组件至少需要以下几个属性：

* **key**：一个用于引用特定场景的唯一字符串。
* **component**：为该 `Scene` 或页面要渲染的组件。
* **title**：显示在屏幕顶部导航栏中的字符串。

需注意，加载的第一个场景有 `initial={true}` 这个属性，以表示它为初始渲染的场景。

<!--more-->

## 页面跳转

### PageOne.js

```js
import React, { Component } from 'react';
import { View, Text } from 'react-native';
import { Actions } from 'react-native-router-flux';

export default class PageOne extends Component {
  render() {
    return (
      <View style={{margin: 128}}>
        <Text onPress={Actions.pageTwo}>This is PageOne!</Text>
      </View>
    )
  }
}
```

从一个页面跳转到另一个页面，必须调用 `Action`。格式为：

```
Actions.SCENE_KEY(PARAMS)
```

`SCENE_KEY` 必须与路由组件根文件其中一个场景定义的 `key` 值匹配。`PARAMS` 指的是一个 javascript 对象，它将作为属性传递到下个场景中（后面会详细介绍）。

由于 PageTwo 组件 key 的值为 `pageTwo`，因此我们需要做的就是将 `Actions.pageTwo`  函数放在 `<Text>` 组件点击事件中，这样在点击文本时便会执行页面跳转。

## 传值

现在尝试扩展示例，以便可以将数据从 `PageOne` 传递到 `PageTwo`。

在 `PageOne.js` 源代码中，用  `Actions.pageTwo({text: 'Hello World!'})` 替换  `Actions.pageTwo`。在这种情况下，需要在函数内部封装 Action 调用，以防止它在渲染这个组件时就执行。因此，`PageOne.js` 中的渲染函数应该如下所示：

```js
render() {
  const goToPageTwo = () => Actions.pageTwo({text: 'Hello World!'}); 
  return (
    <View style={{margin: 128}}>
      <Text onPress={goToPageTwo}>This is PageOne!</Text>
    </View>
  )
}
```

在 `PageTwo.js` 中，可以在现有 `<Text>` 组件下再添加一个 `<Text>` 来使用传入的数据，如下所示：

```js
render() {
  return (
    <View style={{margin: 128}}>
      <Text>This is PageTwo!</Text>
      <Text>{this.props.text}</Text>
    </View>
  )
}
```

现在，跳转到 PageTwo 页面，就应该看到：

```
This is PageTwo!
Hello World!
```

## 向前（或向后？）

介绍了大部分的基础知识，现在迷你教程已结束，但定制 `react-native-router-flux` 还可以做很多，这只是开始。

例如，可能想要以代码方式返回到上一个页面。虽然可以通过点击导航栏左上角的箭头图标来执行此操作，但也可以在应用中的任何位置调用此函数以获得相同的效果：

```js
Actions.pop()
```

甚至可以使用新的参数刷新同一页面：

```js
Actions.refresh(PARAMS)
```


