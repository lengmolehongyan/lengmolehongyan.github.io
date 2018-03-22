---
layout: post
title: "React Components"
date: 2018-03-22 11:19:28 +0800
comments: true
categories: 
keywords: React Components
description: React Components
---

## Component API

### `this.props`

`Components` 组件可以通过将属性传递给构造函数 `constructor` 来实例化配置，这些属性称为 `props`。`props` 可通过组件内函数 `this.props` 来访问，以改变组件的渲染方式或者行为。但是，`props` 不能被组件内函数改变。

父元素可以随时修改子元素 `props`。子元素通常依新配置参数来重新渲染自身。子组件可以决定即使配置发生改变也不重新渲染自身，如由 `shouldComponentUpdate`。

### `this.state`

`Components` 通过 `state` 对象保持它们的状态。在组件内部通过 `this.state` 来访问。不像 `props`，父元素不能访问子元素的 `state`，因为它旨在管理子元素的内部状态而不是外部配置。

注意：不能直接给 `state` 对象赋值，例如 `this.state.foo = 'bar'`，而应该使用 `this.setState()` 函数。

### `this.setState(object newState)`

`Components` 可通过传递对象给 `this.setState()` 函数来更新自身的 `state`。传递给该函数的对象中的任何 key 都将被 `this.state` 合并（并覆盖任何现有的 key）。

## LifeCycle API

组件生命周期：初始化，加载，渲染，更新，卸载，销毁

### Mounting Cycle

* `constructor(object props)`

组件类被初始化。构造函数的参数是元素的初始 `props`，由其父元素指定。可通过给 `this.state` 赋值一个对象来指定元素初始状态。此时，元素的原生 UI 还没有被渲染。

* `componentWillMount()`

此函数只会调用一次，发生在第一次渲染之前。此时元素的原生 UI 同样没被渲染。

* `render() -> React Element`

此函数必须返回一个 React Element 用来渲染（或者 null，不渲染）。

* `componentDidMount()`

此函数只会调用一次，发生在第一次渲染之后。此时元素的原生 UI 已经渲染完毕，可通过 `this.refs` 直接操作。如果需要异步 API 调用或者用 `setTimeout` 执行延迟代码，通常应该在此函数中。

### Updating Cycle

* `componentWillReceiveProps(object nextProps)`

父组件已经传值 `props`。组件将重新渲染。可在 `render` 函数调用前调用 `this.setState()` 来更新组件内部状态。

* `shouldComponentUpdate(object nextProps, object nextState) -> boolean`

基于下一组 `props` 和 `state` 值组件决定是否重新渲染。基类该函数的实现总是返回 `true`（组件应该重新渲染）。为了优化，重写此函数并检查 `props` 或者 `state` 是否被修改，比如运行这些对象中每个键值相等性测试。返回 `false` 将阻止 `render` 函数被调用。

* `componentWillUpdate(object nextProps, object nextState)`

此函数调用时机发生在决定重新渲染之后。不能在这里调用 `this.setState()`，因为已经在更新中。

* `render() -> React Element`

`shouldComponentUpdate` 返回 `true` 此函数才会调用。此函数必须返回一个 React Element 用来渲染（或者 null，不渲染）。

* `componentDidUpdate(object prevProps, object prevState)`

此函数调用时机为重新渲染后。此时，组件的原生 UI 已经更新为 `render()` 函数返回的 React Element。


