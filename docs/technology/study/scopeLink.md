---
sidebar:
  title: 作用域链
#   step: 96
isTimeLine: true
title: 作用域链
date: 2024-06-09
recommend: 5
tags:
  - 技术笔记
categories:
  - 技术笔记
---

# 作用域链

> 当查找变量的时候，会先从当前上下文的变量对象中查找，如果没有找到，就会从父级(词法层面上的父级)执行上下文的变量对象中查找，一直找到全局上下文的变量对象(全局对象)。这样由多个执行上下文的变量对象构成的链表就叫做**作用域链**。

## 函数创建

**函数的作用域在函数定义的时候决定的**
函数有一个内部属性 `[[scope]]`，当函数被创建时，会保存所有父变量对象到其中，可以理解 `[[scope]]` 就是所有父变量对象的层级链
:::warning 提示
`[[scope]]` 并不代表完整的作用域链！

:::

```js
function foo() {
    function bar() {
        ...
    }
}
```

函数创建时，各自的[[scope]]为：

```js
foo.[[scope]] = [
  globalContext.VO
];

bar.[[scope]] = [
    fooContext.AO,
    globalContext.VO
];

```

## 函数被激活

函数被激活时，会创建一个执行上下文，并且将函数的[[scope]]赋值给该上下文的变量对象。

```js
fooContext = {
    AO: {
        arguments: {
            length: 0
        },
    },
    Scope: [AO ...fooScope.[[scope]] ],  // 合并关联
    this: undefined
}
```

### 用于总结的例子

```js
var value = 1;

function foo() {
    console.log(value);
}

function bar() {
    var value = 2;
    foo();
}

bar();

// 函数执行上下文被压入执行上下文栈
ECStack = [
    fooContext
    barContext,
    globalContext
];


// 1. 我的猜想！！！！！！
globalContext = {
    VO: '全部的变量参数',
    Scope: [globalContext.VO],
    this: globalContext.VO
}

barScope.[[scope]] = [
    globalContext.VO
];

fooScope.[[scope]] = [
    barContext.AO,
    globalContext.VO
];


// 执行上下文都有三个属性，变量对象 VO/AO，作用域链 Scope  ，this
// 准备-初始化
fooContext = {
    AO: {
        arguments: {
            length: 0
        },
    },
    Scope: fooScope.[[scope]],
    this: undefined
}

//执行
fooContext = {
    AO: {
        arguments: {
            length: 0
        },
    },
    Scope: [AO ...fooScope.[[scope]] ],  // 合并关联
    this: undefined
}

// 初始化
barContext = {
    AO: {
        arguments: {
            length: 0
        },
        value: undefined,
        foo: reference to function f(){}
    },
    Scope: barScope.[[scope]],
    this: undefined,
}

// 执行
barContext = {
    AO: {
        arguments: {
            length: 0
        },
        value: 2,
        foo: reference to function f(){}
    },
    Scope:[AO ,...barScope.[[scope]]],
    this: undefined
}

```
