---
layout: post
title: "JS 常用代码段"
date: 2020-09-17 17:38:23 +0800
comments: true
categories: 
keywords: "JS,code,snippet"
description: "JS 常用代码段"
---

## 对象

### 判断是否是对象

```javascript
function isObject(obj) {
    return Object.prototype.toString.call(obj) === '[object Object]';
}
```

### 判断对象是否为空

```javascript
function isEmptyObj(obj) {
    return isObject(obj) && Reflect.ownKeys(obj).length === 0;
}

```

## 数组

### 判断是否是数组

```javascript
function isArray(list) {
    if (Array.isArray) {
        return Array.isArray(list);
    }
    return Object.prototype.toString.call(list) === '[object Array]';
}
```

### 判断数组是否为空

```javascript
function isEmptyList(list) {
    return isArray(list) && list.length === 0;
}
```

## 字符串

### 字符串左侧填充

```javascript
String.prototype.padStart()

// syntax
str.padStart(targetLength [, padString])

// example
'abc'.padStart(8, '0'); // "00000abc"
```

### 字符串右侧填充

```javascript
String.prototype.padEnd()

// syntax
str.padEnd(targetLength [, padString])

// example
'abc'.padEnd(6, '123456'); // "abc123"
```